---
title: OpenVPN
author: Joseph Brinkman
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.4
tags:
  - security
  - vpn
---

## 简介

[OpenVPN](https://openvpn.net/) 是一个免费开源的 VPN（Virtual Private Network，虚拟专用网络）。本文将指导您使用 X509 PKI（Public Key Infrastructure，公钥基础设施）设置 OpenVPN。本指南需要一个带有公网 IP 地址的 Rocky Linux 系统，因为 OpenVPN 基于客户端/服务器模型运行。最简单的方法是通过您选择的云提供商启动一个 VPS（Virtual Private Server，虚拟专用服务器）。截至撰写本文时，Google Cloud Platform 为其 e2-micro 实例提供免费层级。如果您正在寻找使用点对点（p2p）VPN 且无 PKI 的最简单 OpenVPN 设置，请参考他们的 [Static Key Mini-HOWTO](https://openvpn.net/community-resources/static-key-mini-howto/)。

## 前提条件与假设

此过程的最低要求如下：

* 能够以 root 用户身份运行命令或使用 `sudo` 提升权限
* 一个具有可公开访问 IP 的 Rocky Linux 系统

## 安装 OpenVPN

安装 EPEL（Extra Packages for Enterprise Linux，企业 Linux 额外软件包）仓库：

```bash
sudo dnf install epel-release -y
```

安装 OpenVPN：

```bash
sudo dnf install openvpn -y
```

## 设置 CA（Certificate Authority，证书颁发机构）

安装 easy-rsa：

```bash
sudo dnf install easy-rsa -y
```

在 `/etc/openvpn` 中创建 `easy-rsa` 目录：

```bash
sudo mkdir /etc/openvpn/easy-rsa
```

创建指向 easy-rsa 文件的符号链接：

```bash
sudo ln -s /usr/share/easy-rsa /etc/openvpn/easy-rsa
```

更改目录到 `/etc/openvpn/easy-rsa`：

```bash
cd /etc/openvpn/easy-rsa
```

运行带有 `init-pki` 参数的 `easyrsa` 脚本以初始化证书颁发机构的 PKI：

```bash
sudo ./easy-rsa/3/easyrsa init-pki
```

运行带有 `build-ca` 和 `nopass` 参数的 `easyrsa` 脚本以构建无需密码的证书颁发机构：

```bash
sudo ./easy-rsa/3/easyrsa build-ca nopass
```

## 创建证书

运行带有 `gen-req` 和 `nopass` 参数的 `easyrsa` 脚本以生成无密码的服务器证书：

```bash
sudo ./easy-rsa/3/easyrsa gen-req server nopass
```

运行带有 `sign-req` 和 `server` 参数的 `easyrsa` 脚本以签署服务器证书：

```bash
sudo ./easy-rsa/3/easyrsa sign-req server server
```

!!! Note

    您可以为额外的客户端根据需要重复以下步骤多次。

运行带有 `gen-req` 和 `nopass` 参数的 `easyrsa` 脚本以生成无密码的客户端证书：

```bash
sudo ./easy-rsa/3/easyrsa gen-req client1 nopass
```

运行带有 `sign-req` 和 `client` 参数的 `easyrsa` 脚本以签署无密码的客户端证书：

```bash
sudo ./easy-rsa/3/easyrsa sign-req client client1
```

OpenVPN 需要 Diffie Hellman 参数。运行此命令生成它们：

```bash
sudo ./easy-rsa/3/easyrsa gen-dh
```

## 配置 OpenVPN

PKI 创建完成后，是时候配置 OpenVPN 了。

将 `server.conf` 示例文件复制到 `/etc/openvpn`：

```bash
sudo cp /usr/share/doc/openvpn/sample/sample-config-files/server.conf /etc/openvpn
```

使用您选择的编辑器打开并写入 `server.conf`：

```bash
sudo vim /etc/openvpn/server.conf
```

接下来，您必须将证书颁发机构、服务器证书和服务器密钥的文件路径添加到 OpenVPN 服务器配置文件中。

在第 78-80 行复制并粘贴密钥和证书的文件路径：

!!! Note

    在 Vim 中，您可以使用 `:set nu` 向当前编辑添加行号

```bash
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key  # This file should be kept secret
```

在示例文件 `server.conf` 的第 85 行复制并粘贴 Diffie Hellman 文件路径：

```bash
dh /etc/openvpn/easy-rsa/pki/dh.pem
```

OpenVPN 默认使用 SSL，但可以选择使用 TLS。本指南使用 SSL。

在第 244 行注释掉 `tls-auth ta.key` 密钥对值：

```bash
#tls-auth ta.key 0 # This file is secret
```

关闭 `server.conf` 之前保存。

## 配置防火墙

OpenVPN 默认在 UDP 端口 1194 上运行。您将使用 `firewalld` 允许 OpenVPN 流量进入服务器。

安装 `firewalld`：

```bash
sudo dnf install firewalld -y
```

启用 `firewalld`：

```bash
sudo systemctl enable --now firewalld
```

通过将其添加为服务来允许 OpenVPN 通过防火墙：

```bash
sudo firewall-cmd --add-service=openvpn --permanent
```

通过向防火墙添加伪装规则来启用 NAT（Network Address Translation，网络地址转换）并隐藏公共客户端 IP 地址：

```bash
sudo firewall-cmd --add-masquerade --permanent
```

重新加载防火墙：

```bash
sudo firewall-cmd --reload
```

## 配置路由

使用以下命令允许 IP 转发：

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

## 启动 OpenVPN 服务器

根据 [OpenVPN 文档](https://openvpn.net/community-resources/how-to/#starting-up-the-vpn-and-testing-for-initial-connectivity)，"最好最初从命令行启动 OpenVPN 服务器"：

```bash
sudo openvpn /etc/openvpn/server.conf
```

启动 OpenVPN 后，按 `Ctrl + Z`，然后将作业发送到后台：

```bash
bg
```

## 配置并启动客户端

除了服务器之外，您还需要在所有客户端上安装 OpenVPN 才能正常工作。如果尚未安装，请在客户端上安装 OpenVPN：

```bash
sudo dnf install openvpn -y
```

创建新目录以存储客户端的密钥、证书和配置文件：

```bash
sudo mkdir -p /etc/openvpn/pki`
```

现在使用安全的传输方法复制密钥和证书，并将它们放在 `/etc/openvpn/pki` 中。您可以这样做的一些潜在方式是使用 SFTP 或 SCP 协议。查看 Rocky Linux 指南 [SSH Public and Private Key](https://docs.rockylinux.org/guides/security/ssh_public_private_keys/) 以设置 SSH 访问。

以下是客户端配置所需的必要证书和密钥及其在服务器上的文件路径：

* ca.crt
* client1.crt
* client1.key

在将必要的证书和密钥存储在 `/etc/openvpn/pki` 后，将示例文件 `client.conf` 复制到 `/etc/openvpn`：

```bash
sudo cp /usr/share/doc/openvpn/sample/sample-config-files/client.conf /etc/openvpn
```

使用您选择的编辑器打开 `client.conf`：

```bash
sudo vim /etc/openvpn/client.conf`
```

将必要证书和密钥的文件路径映射到客户端配置文件。您可以通过在示例文件的第 88-90 行复制粘贴这些文本行来实现：

```bash
ca /etc/openvpn/pki/ca.crt
cert /etc/openvpn/pki/client1.crt
key /etc/openvpn/pki/client1.key
```

您还需要设置服务器主机名或 IP。您可以保留默认的 UDP 端口 1194。在示例文件中，这位于第 42 行：

```bash
remote server 1194
```

退出 `client.conf` 之前保存。

在客户端上启动 OpenVPN：

```bash
sudo openvpn /etc/openvpn/client.conf
```

启动 OpenVPN 后按 `Ctrl + Z` 然后将作业发送到后台：

```bash
bg
```

运行以下命令查看在后台运行的作业：

```bash
jobs
```

向服务器发送测试 ping。默认情况下，其私有地址为 `10.8.0.1`：

```bash
ping 10.8.0.1
```

## 结论

现在您应该拥有自己的 OpenVPN 服务器正在运行！通过此基本配置，您为系统在更大范围的互联网上通信创建了一个安全的私有隧道。然而，OpenVPN 是高度可定制的，本指南留下了很多想象空间。您可以通过查看他们的[网站](https://www.openvpn.net)进一步探索 OpenVPN。您也可以直接在您的系统上阅读更多关于 OpenVPN 的信息 - `man openvpn` - 通过使用手册页。
