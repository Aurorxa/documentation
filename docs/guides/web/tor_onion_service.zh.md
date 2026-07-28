---
title: Tor Onion 服务
author: Neel Chauhan
contributors: Ganna Zhrynova
tested_with: 9.3
tags:
  - web
  - proxy
  - proxies
---

## 简介

[Tor](https://www.torproject.org/) 是一种匿名服务和软件，通过三台由志愿者运行的称为中继 (relays) 的服务器来路由流量。三跳设计旨在通过抵抗监视尝试来确保隐私性。

Tor 的一个功能是你可以运行隐藏的、仅限 Tor 访问的网站，称为 [onion 服务](https://community.torproject.org/onion-services/)。因此，所有到 onion 服务的流量都是私密且加密的。

## 前提条件和假设

以下是使用本步骤的最低要求：

* 能够以 root 用户身份运行命令或使用 `sudo` 提升权限
* 熟悉命令行编辑器。作者在此使用 `vi` 或 `vim`，但可以替换为你喜欢的编辑器
* 一个在 localhost 或其他 TCP/IP 端口上运行的 Web 服务器

## 安装 Tor

要安装 Tor，你需要首先安装 EPEL（企业 Linux 额外软件包）并运行更新：

```bash
dnf -y install epel-release && dnf -y update
```

然后安装 Tor：

```bash
dnf -y install tor
```

## 配置 Tor

软件包安装好后，你需要配置 Tor。作者使用 `vi` 来做这件事，但如果你更喜欢 `nano` 或其他编辑器，尽管替换：

```bash
vi /etc/tor/torrc
```

默认的 `torrc` 文件描述性很好，但如果你只想要一个 onion 服务，它可以变得很长。一个最小的 onion 服务配置类似于：

```bash
HiddenServiceDir /var/lib/tor/onion-site/
HiddenServicePort 80 127.0.0.1:80
```

### 仔细看

* "HiddenServiceDir" 是你 onion 服务的主机名和加密密钥的位置。你将把这些密钥存储在 `/var/lib/tor/onion-site/` 中
* "HiddenServicePort" 是从你的本地服务器到 onion 服务的端口转发。你将 127.0.0.1:80 转发到我们面向 Tor 的服务的端口 80

!!! warning

    如果你计划将你的 onion 服务签名密钥放在 `/var/lib/tor/` 以外的目录中，你需要确保权限是 `0700` 并且所有者是 `toranon:toranon`。

## 配置 Web 服务器

你还需要在我们的机器上有一个 Web 服务器来为你的 onion 服务的客户端提供服务。任何 Web 服务器（Caddy、Apache 或 Nginx）都可以使用。作者更倾向于 Caddy。为简单起见，安装 Caddy：

```bash
dnf -y install caddy
```

接下来，你将以下内容插入到 `/etc/caddy/Caddyfile` 中：

```bash
http:// {
    root * /usr/share/caddy
    file_server
}
```

## 测试并启动

设置好你的 Tor 中继配置后，下一步是启动 Tor 和 Caddy 守护进程：

```bash
systemctl enable --now tor caddy
```

你可以使用此命令获取你的 onion 服务的主机名：

```bash
cat /var/lib/tor/onion-site/hostname
```

在几分钟内，你的 onion 服务将通过 Tor 网络传播，你可以在 Tor 浏览器中查看你的新 onion 服务：

![Tor 浏览器显示我们的 Onion 服务](../images/onion_service.png)

## 结论

Onion 服务在你私下托管网站或仅使用开源软件绕过 ISP 的运营商级 NAT 时是一个宝贵的工具。

虽然 onion 服务不如直接托管网站那么快（鉴于 Tor 以隐私为先的设计，这是可以理解的），但它比公共互联网更加安全和私密。
