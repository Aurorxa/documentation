---
title: Tailscale VPN
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 10.0 
tags:
  - security
  - vpn
---

# Tailscale VPN

## 简介

[Tailscale](https://tailscale.com/) 是一个零配置、端到端加密、基于 Wireguard 的点对点 VPN。Tailscale 支持所有主要的桌面和移动操作系统。

与其他 VPN 解决方案相比，Tailscale 不需要开放 TCP/IP 端口，可以在 NAT（Network Address Translation，网络地址转换）或防火墙后工作。

## 前提条件与假设

使用此过程的最低要求如下：

* 能够以 root 用户身份运行命令或使用 `sudo` 提升权限
* 一个 Tailscale 账户

## 安装 Tailscale

要安装 Tailscale，我们首先需要添加其 `dnf` 仓库：

```bash
dnf config-manager --add-repo https://pkgs.tailscale.com/stable/rhel/10/tailscale.repo
```

然后安装 Tailscale：

```bash
dnf install tailscale
```

## 配置 Tailscale

安装软件包后，您需要启用和配置 Tailscale。要启用 Tailscale 守护进程：

```bash
systemctl enable --now tailscaled
```

随后，您将使用 Tailscale 进行认证：

```bash
tailscale up
```

您将得到一个用于认证的 URL。在浏览器中访问该 URL 并登录 Tailscale：

![Tailscale 登录屏幕](../images/tailscale_1.png)

接下来，您将授权您的服务器访问。点击 **Connect** 来执行操作：

![Tailscale 授权访问对话框](../images/tailscale_2.png)

当您授权访问后，您将看到一个成功对话框：

![Tailscale 登录成功对话框](../images/tailscale_3.png)

一旦您的服务器通过 Tailscale 认证，它将获得一个 Tailscale IPv4 地址：

```bash
tailscale ip -4
```

它还将获得一个 RFC 4193（Unique Local Address，唯一本地地址）Tailscale IPv6 地址：

```bash
tailscale ip -6
```

## 结论

使用 VPN 网关的传统 VPN 服务是集中式的。这需要手动配置、设置防火墙和提供用户账户。Tailscale 通过其点对点模型结合网络级访问控制解决了这个问题。
