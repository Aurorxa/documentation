---
title: OpenBGPD BGP 路由器
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.3
tags:
  - network
---

# OpenBGPD BGP 路由器

## 简介

BGP（边界网关协议）是将互联网连接在一起的路由协议。无论您的互联网服务提供商是谁，正是 BGP 让您能够查看本文档。

[OpenBGPD](http://openbgpd.org/) 是 [OpenBSD](https://www.openbsd.org/) 的跨平台 BGP 实现。作者个人在自己的网络上使用它。

## 前置条件

- 具有 BGP 连接的服务器、虚拟机或实验网络
- 从您的[区域互联网注册机构](https://www.nro.net/about/rirs/) 获取的 AS 号码
- 拥有或租用的 IPv4 或 IPv6 地址块
- 网络管理知识

## 安装软件包

由于 OpenBGPD 不在默认仓库中，请先安装 EPEL 仓库（企业 Linux 额外软件包）：

```bash
dnf install -y epel-release
```

接着安装 OpenBGPD：

```bash
dnf install -y openbgpd
```

## 设置 OpenBGPD

从全新的 OpenBGPD 配置开始：

```bash
rm /etc/bgpd.conf
touch /etc/bgpd.conf
chmod 0600 /etc/bgpd.conf
```

然后将以下内容添加到 `/etc/bgpd.conf`：

```bash
AS YOUR_ASN
router-id YOUR_IPV4

listen on 127.0.0.1
listen on YOUR_IPV4
listen on ::1
listen on YOUR_IPV6
log updates
network IPV4_TO_ADVERTISE/MASK
network IPV6_TO_ADVERTISE/MASK

allow to ebgp prefix { IPV4_TO_ADVERTISE/MASK IPV6_TO_ADVERTISE/MASK }

neighbor PEER_IPV4 {
    remote-as               PEER_ASN
    announce IPv4           unicast
    announce IPv6           none
    local-address           YOUR_IPV4
}

neighbor PEER_IPV6 {
    remote-as               PEER_ASN
    announce IPv4           none
    announce IPv6           unicast
    local-address           YOUR_IPV6
}
```

替换以下信息：

- **YOUR_ASN** 替换为您的 AS 号码。
- **YOUR_IPV4** 替换为服务器的 IPv4 地址。
- **YOUR_IPV6** 替换为服务器的 IPv6 地址。
- **PEER_ASN** 替换为上游 ISP 的 AS 号码。
- **PEER_IPV4** 替换为上游 ISP 的 IPv4 地址。
- **PEER_IPV6** 替换为上游 ISP 的 IPv6 地址。

上述各行的含义如下：

- `AS` 行包含您的 BGP AS 号码。
- `router-id` 行包含您的 BGP 路由器 ID。这是一个 IPv4 地址，但如果您只进行纯 IPv6 BGP，可以使用虚拟不可路由地址（如 169.254.x.x）。
- `listen on` 行指定要监听的接口。我们应当在所有运行 BGP 的接口上监听。
- `network` 行添加我们要宣告的网络。
- `allow to ebgp prefix` 行添加 [RFC8212](https://datatracker.ietf.org/doc/html/rfc8212) 合规性以增强路由安全。某些托管公司（如 BuyVM）要求此项。
- `neighbor` 块分别指定每个 IPv4 和 IPv6 对等。
- `remote-as` 行指定上游的 AS 号码。
- `announce IPv4` 行指定是否宣告 `unicast` IPv4 路由或 `none`。IPv6 上游应设为 `none`。
- `announce IPv6` 行指定是否宣告 `unicast` IPv6 路由或 `none`。IPv4 上游应设为 `none`。
- `local-address` 行是上游的 IPv4 或 IPv6 地址。

某些上游可能使用 MD5 密码或 BGP multihop。如果是这种情况，您的 `neighbor` 块将如下所示：

```bash
neighbor PEER_IPV4 {
    remote-as               PEER_ASN
    announce IPv4           unicast
    announce IPv6           none
    local-address           YOUR_IPV4
    multihop                2
    local-address           203.0.113.123
}

neighbor PEER_IPV6 {
    remote-as               PEER_ASN
    announce IPv4           none
    announce IPv6           unicast
    local-address           YOUR_IPV6
    multihop                2
    local-address           2001:DB8:1000::1
}
```

您需要通过设置以下 `sysctl` 值来启用 IP 转发：

```bash
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```

现在，启用 OpenBGPD 和转发功能：

```bash
sysctl -p /etc/sysctl.conf
systemctl enable --now bgpd
```

## 检查 BGP 状态

OpenBGPD 启用后，可以查看 BGP 状态：

```bash
bgpctl show
```

您将看到如下输出：

```bash
Neighbor                   AS    MsgRcvd    MsgSent  OutQ Up/Down  State/PrfRcvd
BGP_PEER             PEER_ASN       164         68     0 00:32:04      0
```

也可以查看 BGP 宣告的路由：

```bash
bgpctl show rib
```

如果一切正常，您应该能看到 BGP 路由表：

```bash
flags: * = Valid, > = Selected, I = via IBGP, A = Announced,
       S = Stale, E = Error
origin validation state: N = not-found, V = valid, ! = invalid
aspa validation state: ? = unknown, V = valid, ! = invalid
origin: i = IGP, e = EGP, ? = Incomplete

flags  vs destination          gateway          lpref   med aspath origin
AI*>  N-? YOUR_IPV4/24         0.0.0.0           100     0 i
AI*>  N-? YOUR_IPV6::/48       ::                100     0 i
```

## 总结

虽然 BGP 起初可能看起来令人生畏，但一旦掌握，您就可以获得属于自己的一片互联网路由表。OpenBGPD 的简洁性使得拥有软件路由器或 anycast 服务器变得更加容易。享受吧！
