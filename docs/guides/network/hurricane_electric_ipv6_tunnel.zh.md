---
title: Hurricane Electric IPv6 隧道
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.5
tags:
  - network
---

# Hurricane Electric IPv6 隧道

IPv6 无需过多介绍，但如果您不了解，它是取代更流行的 IPv4 协议的下一代协议，使用 128 位十六进制地址而非 32 位十进制地址。

[Hurricane Electric](https://he.net) 是一家互联网服务提供商。在其众多服务中，Hurricane Electric 提供免费的[隧道代理 (Tunnel Broker)](https://tunnelbroker.net/) 服务，为仅支持 IPv4 的网络提供 IPv6 连接。

## 简介

由于 IPv4 地址枯竭，需要以 IPv6 的形式提供扩展的 IP 寻址空间。然而，由于 NAT（网络地址转换）的普遍存在，许多网络仍然缺乏 IPv6 支持。正因如此，Hurricane Electric 提供了 IPv6 隧道。

## 前置条件

- 一个[免费的 Hurricane Electric IPv6 隧道](https://tunnelbroker.net/)

- 一台拥有公网 IP 地址且未过滤 ICMP（互联网控制消息协议）的 Rocky Linux 服务器。

## 获取 IPv6 隧道

首先，在 [tunnelbroker.net](https://tunnelbroker.net/) 上创建账户。

拥有账户后，在 **User Functions** 侧边栏中选择 **Create Regular Tunnel**：

![HE.net 侧边栏](../images/henet_1.png)

然后输入您的公网 IPv4 地址，选择您的终端节点位置，点击 **Create Tunnel**。

## 设置 IPv6 隧道

好消息是，IPv6 隧道只需要一条命令：

```bash
nmcli connect add type ip-tunnel ifname he-sit mode sit remote IPV4_SERVER ipv4.method disabled ipv6.method manual ipv6.address IPV6_CLIENT ipv6.gateway IPV6_SERVER
```

使用 Hurricane Electric 门户中的详细信息替换以下内容：

- `IPV4_SERVER` 替换为 **Server IPv4 Address**
- `IPV6_SERVER` 替换为 **Server IPv6 Address**
- `IPV6_CLIENT` 替换为 **Client IPv6 Address**
