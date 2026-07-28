---
title: NSD 权威 DNS 服务器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - dns
  - nsd
---

# 权威 DNS 服务器 —— NSD

## 引言

NSD (Name Server Daemon) 是一个高性能的权威 DNS 服务器(authoritative-only DNS server)，由 NLnet Labs 开发。它与 Knot 类似，但各有千秋。NSD 是为性能而设计的权威专用 DNS 服务器。它通过将 DNS 区域预编译为数据库文件来优化性能。

本文将引导您完成在 Rocky Linux 上安装和配置 NSD DNS 服务器的过程。

您将构建一个虚构网络。该网络的主要 DNS 服务器是 `ns1.example.com`，它支持位于 192.168.0.0/24 私有(Private) C 类网络中的邮件和 Web 服务器。在此示例中，这两个服务器的 IP 地址分别对应 `192.168.0.13` 和 `192.168.0.14`。

## 前提条件

* 两台运行 Rocky Linux 的服务器（主要 DNS 服务器 `ns1.example.com` 和 `ns2.example.com`）
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验
* 对 DNS 有基本了解

## 安装 EPEL 和 NSD

NSD 在 Rocky Linux 中默认不可用，需要通过 EPEL（Extra Packages for Enterprise Linux，企业 Linux 额外软件包）仓库进行安装。

```bash
dnf install -y epel-release
dnf install -y nsd
```

## 配置服务器

`nsd` 软件包会将默认配置文件安装在 `/etc/nsd/nsd.conf` 中。该文件仅用于基本配置。您需要将其`备份`并创建自己的配置。

```bash
cp /etc/nsd/nsd.conf /etc/nsd/nsd.conf.orig
```

区域数据(zone data)将存储在 `/var/lib/nsd/` 目录中，因为这是 `nsd` 服务默认的工作目录。当然，您可以使用其他路径，只需记得在配置中指定即可。对于本指南，我们使用默认位置。

### 命名约定

本指南使用以下命名约定来指示哪些区域文件被用于正向查找（名称转 IP）和反向查找（IP 转名称）。

| 名称                     | 描述                                   |
|--------------------------|----------------------------------------|
| example.com.zone         | 正向查找文件名称                       |
| 0.168.192.in-addr.arpa.zone | 反向查找文件名称                       |
| localhost.zone           | 本地主机正向查找                       |
| 0.0.127.in-addr.arpa.zone   | 本地主机反向查找                       |

### 配置主服务器 `ns1`

主要配置选项分为 `server` 和 `zone` 部分。服务器配置的示例如下。请将示例中的地址 `192.168.0.10` 替换为您实际的主要 DNS 服务器的 IP 地址：

```text
server:
    server-count: 1
    username: nsd
    zonesdir: "/var/lib/nsd"
    logfile: "/var/log/nsd.log"
    pidfile: "/run/nsd/nsd.pid"
    ip-address: 192.168.0.10
    port: 53
```

* `server-count` 指定要启动的 NSD 服务器进程数量。如果服务器像本指南中那样有多个 IP 地址，可能会有用。

现在我们来配置区域(zone)。区域的格式如下：

```text
zone:
    name: "example.com"
    zonefile: "example.com.zone"
    notify: 192.168.0.11 NOKEY
    provide-xfr: 192.168.0.11 NO_KEY

zone:
    name: "0.168.192.in-addr.arpa"
    zonefile: "0.168.192.in-addr.arpa.zone"
```

* `name` 是需要使用的 zone 名称。
* `zonefile` 是 zone 配置文件的路径。
* `notify` 是可选的，指定当 zone 发生变化时通知从服务器。格式为 `notify: <ip-address> <key-name>`。
* `provide-xfr` 用于允许区域传输(zone transfer)到从服务器。

接下来，创建 `example.com.zone` 正向区域文件。该文件包含区域信息，并以 BIND 格式编写。

```text
$ORIGIN example.com.
$TTL 86400
             3600      SOA       ns1.example.com. admin.example.com. (
                                2022101401 ; 序列号
                                3600       ; 刷新间隔
                                1800       ; 重试间隔
                                604800     ; 过期时间
                                86400      ; 最小 TTL
                                )
             3600      NS        ns1.example.com.
             3600      NS        ns2.example.com.
             3600      MX 10     mail.example.com.
             3600      A         192.168.0.13
ns1                    A         192.168.0.10
ns2                    A         192.168.0.11
mail                   A         192.168.0.13
www                    A         192.168.0.14
```

至此，服务器配置完成。现在启动 `nsd` 守护进程：

```bash
systemctl enable --now nsd
```

### 配置从服务器 `ns2`

从服务器 `ns2` 的配置与 `ns1` 非常相似，主要区别在于 IP 地址和区域配置部分。

从服务器的 `zone` 配置部分如下所示：

```text
zone:
    name: "example.com"
    zonefile: "example.com.zone"
    allow-notify: 192.168.0.10 NOKEY
    request-xfr: 192.168.0.10 NOKEY
```

启动从服务器的 `nsd` 服务：

```bash
systemctl enable --now nsd
```

## 配置防火墙

您需要确保防火墙允许 DNS 查询通过。根据您使用的防火墙工具，配置方法如下：

`firewalld`：

```bash
firewall-cmd --add-service=dns --permanent
firewall-cmd --reload
```

`iptables`：

```bash
iptables -A INPUT -p tcp --dport 53 -j ACCEPT
iptables -A INPUT -p udp --dport 53 -j ACCEPT
service iptables save
```

## 测试

### 检查 `nsd` 运行状态

```bash
systemctl status nsd
```

### 检查区域传输

使用 `dig` 命令请求区域传输来测试服务器：

```bash
dig @ns1.example.com example.com AXFR
```

此命令应该从主服务器返回完整的区域数据。

使用 `nslookup`：

```bash
nslookup mail.example.com
```

## 结论

您现在拥有一个功能完备的权限 DNS 服务器，它采用了主/从拓扑结构。NSD 是一个高性能、专门优化过的权威 DNS 服务器，适合需要高性能、专门化 DNS 服务的基础架构部署。
