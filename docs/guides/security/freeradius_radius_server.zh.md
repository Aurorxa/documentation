---
title: FreeRADIUS RADIUS 服务器
author: Neel Chauhan
contributors: Steven Spencer
tested_with: 9.4
tags:
  - security
---

# FreeRADIUS 802.1X 服务器

## 简介

RADIUS 是一个 AAA（认证、授权和计费，authentication, authorization and accounting）协议，用于管理网络访问。[FreeRADIUS](https://www.freeradius.org/) 是 Linux 和其他类 Unix 系统上事实上的 RADIUS 服务器。

## 前提条件与假设

此过程的最低要求如下：

* 能够以 root 用户身份运行命令或使用 `sudo` 提升权限
* 一个 RADIUS 客户端，例如路由器、交换机或 Wi-Fi 接入点

## 安装 FreeRADIUS

您可以从 `dnf` 仓库安装 FreeRADIUS：

```bash
dnf install -y freeradius
```

## 配置 FreeRADIUS

安装软件包后，您需要首先为 FreeRADIUS 生成 TLS 加密证书：

```bash
cd /etc/raddb/certs
./bootstrap
```

随后，您将需要添加要认证的用户。打开 `users` 文件：

```bash
cd ..
vi users
```

在文件中，插入以下内容：

```bash
user    Cleartext-Password := "password"
```

将 `user` 和 `password` 分别替换为所需的用户名和密码。

请注意密码未经过哈希处理，因此如果攻击者获取了 `users` 文件，他们可能会未经授权访问您受保护的网络。

您也可以使用 `MD5` 哈希或 `Crypt` 哈希密码。要生成 MD5 哈希密码，运行：

```bash
echo -n password | md5sum | awk '{print $1}'
```

将 `password` 替换为所需的密码。

您将得到一个哈希值 `5f4dcc3b5aa765d61d8327deb882cf99`。在 `users` 文件中，改为插入以下内容：

```bash
user    MD5-Password := "5f4dcc3b5aa765d61d8327deb882cf99"
```

您还需要定义客户端。这是为了防止未经授权访问我们的 RADIUS 服务器。编辑 `clients.conf` 文件：

```bash
vi clients.conf
```

插入以下内容：

```bash
client 172.20.0.254 {
        secret = secret123
}
```

将 `172.20.0.254` 和 `secret123` 替换为客户端将使用的 IP 地址和密钥值。为其他客户端重复此操作。

## 启用 FreeRADIUS

初始配置后，您可以启动 `radiusd`：

```bash
systemctl enable --now radiusd
```

## 在交换机上配置 RADIUS

设置 FreeRADIUS 服务器后，您将配置 RADIUS 客户端。

作为示例，作者的 MikroTik 交换机可以这样配置：

```bash
/radius
add address=172.20.0.12 secret=secret123 service=dot1x
/interface dot1x server
add interface=combo3
```

将 `172.20.0.12` 替换为 FreeRADIUS 服务器的 IP 地址，将 `secret123` 替换为您之前设置的密钥。
