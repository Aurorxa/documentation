---
title: accel-ppp PPPoE 服务器
author: Neel Chauhan
contributors:
tested_with: 9.3
tags:
  - network
---

# accel-ppp PPPoE 服务器

## 简介

PPPoE 是一种主要由 DSL 和光纤到户 ISP 使用的协议，通过用户名和密码组合对客户进行身份验证。PPPoE 适用于那些要求现有 ISP 与其他 ISP 共享网络的国家，因为客户可以通过域名路由到所需的 ISP。

[accel-ppp](https://accel-ppp.org/) 是 PPPoE 及相关协议（如 PPTP、L2TP 等）的 Linux 内核加速实现。

## 前置条件

- 一台具有两个网络接口的服务器
- 一台支持 PPPoE 的客户端路由器或机器

## 安装 accel-ppp

首先安装 EPEL：

```bash
dnf install -y epel-release
```

接着安装 accel-ppp：

```bash
dnf install -y accel-ppp
```

## 设置 accel-ppp

首先，需要启用 IP 转发：

```bash
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p
```

然后将以下内容添加到 `/etc/accel-ppp.conf`：

```bash
[modules]
log_file
pppoe
auth_mschap_v2
auth_mschap_v1
auth_chap_md5
auth_pap
chap-secrets
ippool

[core]
log-error=/var/log/accel-ppp/core.log
thread-count=4

[ppp]
ipv4=require

[pppoe]
interface=YOUR_INTERFACE

[dns]
dns1=YOUR_DNS1
dns2=YOUR_DNS2

[ip-pool]
gw-ip-address=YOUR_GW
YOUR_IP_RANGE

[chap-secrets]
gw-ip-address=YOUR_GW
chap-secrets=/etc/chap-secrets
```

替换以下信息：

- **YOUR_INTERFACE** 替换为监听 PPPoE 客户端的接口。
- **YOUR_DNS1** 和 **YOUR_DNS2** 替换为要分发给客户端的 DNS 服务器。
- **YOUR_GW** 是服务器的 IP 地址，用于 PPPoE 客户端。此地址**必须**与服务器的 WAN 侧 IP 地址或默认网关不同。
- **YOUR_IP_RANGE** 替换为要分发给客户端的 IP 范围。可以是 IP 范围（如 X.X.X.Y-Z）或 CIDR 格式（如 X.X.X.X/MASK）。

接着，添加一个基础的 `/etc/chap-secrets` 文件：

```bash
user	*	password	*
```

可以通过添加更多行来添加更多用户，将 `user` 和 `password` 替换为所需的用户名和密码。

## 配置 PPPoE 客户端

PPPoE 服务器设置完成后，就可以开始添加 PPPoE 客户端了。作者喜欢使用 [MikroTik CHR](https://help.mikrotik.com/docs/display/ROS/Cloud+Hosted+Router%2C+CHR) 作为首选的测试 PPPoE 客户端，这里也以此为例。

在连接到与 PPPoE 服务器监听接口相同以太网络的系统上安装 MikroTik CHR 后，配置 PPPoE：

```bash
[admin@MikroTik] > /interface pppoe-client
[admin@MikroTik] > add add-default-route=yes disabled=no interface=ether1 name=pppoe-out1 \
    password=password user=user
```

如果一切正常，应该会获得一个 IPv4 地址：

```bash
[admin@MikroTik] > /ip/address/print
Flags: D - DYNAMIC
Columns: ADDRESS, NETWORK, INTERFACE
#   ADDRESS      NETWORK   INTERFACE 
0 D 10.0.0.1/32  10.0.0.0  pppoe-out1
```

## 总结

PPPoE 经常受到诟病，原因显而易见：需要手动配置用户名和密码。尽管如此，在 ISP 场景中，当连接到二层广播域时，如果需要安全性，但又不希望使用 802.1X 或 MACsec（例如为了允许客户自备路由器或静态 IP 地址），PPPoE 确实提供了安全保障。现在您已经是自己的迷你 ISP 了，恭喜！
