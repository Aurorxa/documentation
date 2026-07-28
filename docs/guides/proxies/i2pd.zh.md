---
title: i2pd 匿名网络
author: Neel Chauhan
contributors: Steven Spencer
tags:
  - proxy
  - proxies
---

## 简介

[I2P](https://geti2p.net/en/) 是一个匿名覆盖网络，与更流行的 Tor 网络竞争，专注于称为 eepsites 的隐藏网站。[`i2pd`](https://i2pd.website/)（I2P Daemon）是 I2P 协议的轻量级 C++ 实现。

## 前提条件与假设

使用此过程的最低要求如下：

- 一个公网 IPv4 或 IPv6 地址，无论是在服务器上直接拥有，还是通过端口转发或 UPnP/NAT-PMP 方式

## 安装 `i2pd`

要安装 `i2pd`，您需要首先安装 EPEL（Extra Packages for Enterprise Linux，企业 Linux 额外软件包）和 `i2pd` copr（Cool Other Package Repo）仓库：

```bash
curl -s https://copr.fedorainfracloud.org/coprs/supervillain/i2pd/repo/epel-10/supervillain-i2pd-epel-10.repo -o /etc/yum.repos.d/i2pd-epel-10.repo
dnf install -y epel-release
```

然后安装 `i2pd`：

```bash
dnf install -y i2pd
```

## 配置 `i2pd`（可选）

安装软件包后，您可以根据需要配置 `i2pd`。作者使用 `vim` 进行此操作，但如果您更喜欢 `nano` 或其他编辑器，可以替换使用：

```bash
vim /etc/i2pd/i2pd.conf
```

默认的 `i2pd.conf` 文件描述性很强，但如果您只想要基本配置，可以保持原样。

但是，如果您想启用 IPv6 和 UPnP 并将监听 HTTP 代理端口设置为 `12345`，可以使用以下配置：

```bash
ipv6 = true
[httpproxy]
port = 12345
[upnp]
enabled = true
```

如果您想设置其他选项，配置文件中对所有可能的选项都有详细的说明。

## 启用 `i2pd`

现在我们可以启用 `i2pd`

```bash
systemctl enable --now i2pd
```

## 访问 I2P eepsites

本示例在 Rocky Linux 上使用 Firefox。如果您不使用 Firefox，请参阅您的应用程序文档来设置 HTTP 代理。

打开 Firefox，点击汉堡菜单图标，然后转到**设置**：

![Firefox 菜单下拉](../images/i2p_proxy_ff_1.png)

滚动到**网络设置**并随后按下**设置**：

![Firefox 网络设置部分](../images/i2p_proxy_ff_2.png)

然后选择**手动代理配置**，输入 `localhost` 和 `4444`（或您选择的端口），勾选**也使用此代理进行 HTTPS**并选择**确定**。

![Firefox 连接设置对话框](../images/i2p_proxy_ff_3.png)

现在，您可以浏览 I2P eepsites 了。作为示例，导航到 `http://planet.i2p`（注意：`http://` 很重要，以防止 Firefox 默认使用搜索引擎）：

![Firefox 查看 planet.i2p](../images/i2p_proxy_ff_4.png)

## 结论

由于许多互联网用户关注在线隐私，I2P 是一种安全访问隐藏网站的方式。`i2pd` 是轻量级软件，使浏览 I2P 网站成为可能，同时还可以共享您的连接作为中继。
