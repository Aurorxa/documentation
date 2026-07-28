---
title: WireGuard VPN
author: Joseph Brinkman
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.4
tags:
  - security
  - vpn
---

## 简介

[WireGuard](https://www.wireguard.com/) 是一个免费开源的 P2P（Peer-to-Peer，点对点）VPN（Virtual Private Network，虚拟专用网络）。它是一个轻量级且安全的现代替代方案，替代了那些依赖 TCP 连接、具有大型代码库的传统 VPN。由于 WireGuard 是 P2P VPN，每个加入到 WireGuard 网络的计算机彼此直接通信。本指南使用中心-分支（hub-spoke）模型，将一个分配了公网 IP 地址的 WireGuard 对等节点作为网关来传递所有流量。这使得 WireGuard 流量能够在无需在路由器上启用端口转发的情况下绕过 CGNAT（Carrier Grade NAT，运营商级 NAT）。这需要一个带有公网 IP 地址的 Rocky Linux 系统。最简单的方法是通过您选择的云提供商启动一个 VPS（Virtual Private Server，虚拟专用服务器）。截至撰写本文时，Google Cloud Platform 为其 e2-micro 实例提供免费层级。

## 前提条件与假设

此过程的最低要求如下：

* 能够以 root 用户身份运行命令或使用 `sudo` 提升权限
* 一个具有可公开访问 IP 地址的 Rocky Linux 系统

## 安装 WireGuard

安装 EPEL（Extra Packages for Enterprise Linux，企业 Linux 额外软件包）：

```bash
sudo dnf install epel-release -y
```

升级系统软件包：

```bash
sudo dnf upgrade -y
```

安装 WireGuard：

```bash
sudo dnf install wireguard-tools -y
```

## 配置 WireGuard 服务器

设置一个临时掩码，使只有所有者对新创建的文件有读写访问权限。

!!! Note

    该掩码仅适用于当前终端会话。一旦您关闭窗口或注销，系统默认值将会恢复。

```bash
umask 077
```

创建一个文件夹来存放您的 WireGuard 配置文件和密钥：

```bash
sudo mkdir -p /etc/wireguard
```

创建一个以 `.conf` 扩展名结尾的、您命名的配置文件：

!!! Note

    您可以在同一台机器上创建多个 WireGuard VPN 隧道，每个使用不同的配置文件、网络地址和 UDP 端口。

```bash
sudo touch /etc/wireguard/wg0.conf
```

为 WireGuard 服务器生成新的私钥和公钥对：

```bash
wg genkey | sudo tee /etc/wireguard/wg0 | wg pubkey | sudo tee /etc/wireguard/wg0.pub
```

使用您选择的编辑器编辑配置文件。

```bash
sudo vi /etc/wireguard/wg0.conf
```

粘贴以下内容：

```bash
[Interface]
PrivateKey = server_privatekey
Address = x.x.x.x/24
ListenPort = 51820
```

您必须用之前生成的私钥替换 `server_privatekey`。您可以使用以下命令查看私钥：

```bash
sudo cat /etc/wireguard/wg0
```

接下来，您需要用 [RFC 1918](https://datatracker.ietf.org/doc/html/rfc1918) 定义的私有 IP 地址范围内的网络地址替换 `x.x.x.x/24`。本指南中使用的网络地址是 `10.255.255.0/24`。

最后，您可以选择任何 UDP 端口来接受 WireGuard VPN 的连接。本指南使用 UDP 端口 `51820`。  

## 启用 IP 转发

IP 转发允许在网络之间路由数据包。这使得内部设备可以通过 WireGuard 隧道相互通信：

开启 IPv4 和 IPv6 的 IP 转发：

```bash
sudo sysctl -w net.ipv4.ip_forward=1 && sudo sysctl -w net.ipv6.conf.all.forwarding=1
```

## 配置 `firewalld`

安装 `firewalld`：

```bash
sudo dnf install firewalld -y
```

安装 `firewalld` 后，启用它：

```bash
sudo systemctl enable --now firewalld
```

创建一个永久防火墙规则，允许在 public 区域中 UDP 端口 51820 上的流量：

```bash
sudo firewall-cmd --permanent --zone=public --add-port=51820/udp
```

接下来，来自 WireGuard 接口的流量将被允许到 internal 区域中的其他接口。

```bash
sudo firewall-cmd --permanent --add-interface=wg0 --zone=internal
```

添加防火墙规则以在内部流量上启用 IP 伪装。这意味着对等节点之间发送的数据包将把数据包 IP 地址替换为服务器的 IP 地址：

```bash
sudo firewall-cmd --permanent --zone=internal --add-masquerade
```

最后，重新加载 `firewalld`：

```bash
sudo firewall-cmd --reload
```

## 配置 WireGuard 对等节点

由于 WireGuard 网络中的所有计算机在技术上是平等的，此过程与配置 WireGuard 服务器几乎相同，但略有不同。

创建一个文件夹来存放您的 WireGuard 配置文件和密钥：

```bash
sudo mkdir -p /etc/wireguard
```

创建一个以 `.conf` 扩展名结尾的、您命名的配置文件：

```bash
sudo touch /etc/wireguard/wg0.conf
```

生成新的私钥和公钥对：

```bash
wg genkey | sudo tee /etc/wireguard/wg0 | wg pubkey | sudo tee /etc/wireguard/wg0.pub
```

使用您选择的编辑器编辑配置文件，添加以下内容：

```bash
[Interface]
PrivateKey = peer_privatekey
Address = 10.255.255.2/24

[Peer]
PublicKey = server_publickey
AllowedIPs = 10.255.255.1/24
Endpoint = serverip:51820
PersistentKeepalive = 25
```

将对等节点的私钥替换为存储在对等节点上 `/etc/wireguard/wg0` 中的 `peer_privatekey`。

您可以使用此命令输出密钥以便复制：

```bash
sudo cat /etc/wireguard/wg0
```

将服务器的公钥替换为存储在服务器上 `/etc/wireguard/wg0.pub` 中的 `server_publickey`。

您可以使用此命令输出密钥以便复制：

```bash
sudo cat /etc/wireguard/wg0.pub
```

将 `serverip` 替换为 WireGuard 服务器的公网 IP。

您可以在服务器上使用以下命令查找服务器的公网 IP 地址：

```bash
ip a | grep inet
```

对等节点的配置文件现在包含一个 `PersistentKeepalive = 25` 规则。此规则告诉对等节点每 25 秒 ping WireGuard 服务器一次以保持 VPN 隧道的连接。没有此设置，VPN 隧道将在不活动后超时。

## 启用 WireGuard VPN

要启用 WireGuard，您将在服务器和对等节点上都运行以下命令：

```bash
sudo systemctl enable wg-quick@wg0
```

然后通过在服务器和对等节点上都运行此命令来启动 VPN：

```bash
sudo systemctl start wg-quick@wg0
```

## 将客户端密钥添加到 WireGuard 服务器配置

输出对等节点的公钥并复制它：

```bash
sudo cat /etc/wireguard/wg0.pub
```

在服务器上，运行以下命令，用对等节点的公钥替换 `peer_publickey`：

```bash
sudo wg set wg0 peer peer_publickey allowed-ips 10.255.255.2
```

使用 `wg set` 只对 WireGuard 接口进行临时更改。对于永久配置更改，您可以手动编辑配置文件并添加对等节点。在进行任何永久配置更改后，您需要重新加载 WireGuard 接口。

使用您选择的编辑器编辑服务器的配置文件。

```bash
sudo vi /etc/wireguard/wg0.conf
```

将对等节点添加到配置文件。内容应该类似于下面所示：

```bash
[Interface]
PrivateKey = +Eo5oVjt+d3XWvFWYcOChaLroGj5vapdXKH8UZ2T2Fc=
Address = 10.255.255.1/24
ListenPort = 51820

[Peer]
PublicKey = 1vSho8NvECkG1PVVk7avZWDmrd2VGZ2xTPaNe5+XKSg=
AllowedIps = 10.255.255.2/32
```

关闭接口：

```bash
sudo wg-quick down wg0
```

启动接口：

```bash
sudo wg-quick up wg0
```

## 查看 WireGuard 接口并测试连接

您可以在服务器和对等节点上使用以下命令查看 WireGuard 信息：

```bash
sudo wg
```

您可以通过从对等节点发送 ping 到服务器来测试连接性：

```bash
ping 10.255.255.1
```

## 结论

按照本指南，您已成功使用 hub-spoke 模型设置了 WireGuard VPN。此配置提供了一种安全、现代且高效的方式，在互联网上连接多个设备。请查阅[官方 WireGuard 网站](https://www.wireguard.com/)。
