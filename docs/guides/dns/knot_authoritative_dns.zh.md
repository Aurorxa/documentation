---
title: Knot 权威 DNS 服务器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - dns
  - knot
---

# 权威 DNS 服务器 —— Knot

## 引言

Knot DNS 是一个高性能的权威 DNS 服务器(authoritative-only DNS server)，支持所有关键的 DNS 协议，包括向从服务器(slave)进行的区域传输(zone transfer)、动态区域更新和 NOTIFY 等。它允许及时分发新的或更改过的区域。本文档将引导您完成在 Rocky Linux 上安装和配置 Knot DNS 服务器的过程。

您将构建一个虚构网络。该网络的主要 DNS 服务器是 `ns1.example.com`，它支持位于 192.168.0.0/24 私有(Private)C 类网络中的邮件和 Web 服务器。在此示例中，这两个服务器的 IP 地址分别对应 `192.168.0.13` 和 `192.168.0.14`。

## 前提条件

* 两台运行 Rocky Linux 的服务器（主要 DNS 服务器 `ns1.example.com` 和 `ns2.example.com`）
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验
* 对 DNS 有基本了解

## 安装 EPEL 和 Knot DNS

Knot DNS 在 Rocky Linux 中默认不可用，需要通过 EPEL（Extra Packages for Enterprise Linux，企业 Linux 额外软件包）仓库进行安装。

```bash
dnf install -y epel-release
dnf install -y knot
```

## 配置服务器

`knot` 软件包会将默认配置文件安装在 `/etc/knot/knot.conf` 中。该文件仅用于基本配置。您需要将其`备份`并创建自己的配置。

```bash
cp /etc/knot/knot.conf /etc/knot/knot.conf.orig
```

区域数据（zone data）将存储在 `/var/lib/knot/` 目录中，因为这是 `knot` 服务默认的工作目录。当然，您可以使用其他路径，只需记得在配置中指定即可。对于本指南，我们使用默认位置。

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
    rundir: /run/knot
    user: knot:knot
    listen: 192.168.0.10@53
    listen: ::@53
    identity: ns1.example.com
    nsid: ns1_example_com
log:
  - target: syslog
    any: info
```

* `rundir` 指定 knot 存储运行时数据的目录。
* `user` 指定 knot 进程运行的用户。
* `listen` 用于配置 knot 监听的地址和端口。这里使用 `53` 端口，因为它是 DNS 的默认端口。如果需要，您可以在 IP 地址后添加 `@` 并指定端口。
* `identity` 返回服务器 ID。
* `nsid` 用于 DNS 名称服务器标识 (RFC 5001)。
* `log` 启用 Knot 的系统日志记录。

现在我们来配置区域(zone)。区域的格式如下：

```text
zone:
  - domain: example.com
    file: /var/lib/knot/example.com.zone
    notify: ns2.example.com
    acl: [net_acl, transfers]
  - domain: 0.168.192.in-addr.arpa
    file: /var/lib/knot/0.168.192.in-addr.arpa.zone
    notify: ns2.example.com
    acl: [net_acl, transfers]
  - domain: localhost
    file: /var/lib/knot/localhost.zone
    notify: ns2.example.com
    acl: [net_acl, transfers]
  - domain: 0.0.127.in-addr.arpa
    file: /var/lib/knot/0.0.127.in-addr.arpa.zone
    notify: ns2.example.com
    acl: [net_acl, transfers]
```

* `domain` 是需要使用的 zone 名称。
* `file` 是 zone 配置文件的路径。
* `notify` 是可选的，指定当 zone 发生变化时通知从服务器。
* `acl` 指定访问控制列表(ACL)，用于限制哪些客户端可以查询或接收区域传输(zone transfer)。ACLs 使用 `remotes` 模块进行定义。

```text
acl:
  - id: net_acl
    address: 192.168.0.0/24
    action: notify
  - id: transfers
    address: 192.168.0.11
    action: transfer
```

* `action: notify` 通常设置为通知从服务器。而 `action: transfer` 则用于允许区域传输。

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

至此，服务器配置完成。现在启动 `knot` 守护进程：

```bash
systemctl enable --now knot
```

### 配置从服务器 `ns2`

从服务器 `ns2` 的配置与 `ns1` 非常相似，主要区别在于 `acl` 段和区域配置部分。

从服务器的 `zone` 配置部分如下所示：

```text
zone:
  - domain: example.com
    file: /var/lib/knot/example.com.zone
    master: ns1.example.com
    acl: net_acl
  - domain: 0.168.192.in-addr.arpa
    file: /var/lib/knot/0.168.192.in-addr.arpa.zone
    master: ns1.example.com
    acl: net_acl
  - domain: localhost
    file: /var/lib/knot/localhost.zone
    master: ns1.example.com
    acl: net_acl
  - domain: 0.0.127.in-addr.arpa
    file: /var/lib/knot/0.0.127.in-addr.arpa.zone
    master: ns1.example.com
    acl: net_acl
```

对于 `acl` 部分，直接定义 `net_acl` 即可，因为这是唯一用于检索区域数据的地址。无需 `transfers` ACL。

启动从服务器的 `knot` 服务：

```bash
systemctl enable --now knot
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

### 检查 `knot` 运行状态

```bash
systemctl status knot
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

您现在拥有一个功能完备的权限 DNS 服务器，它采用了主/从拓扑结构。Knot DNS 是一个高性能、现代化的权威 DNS 服务器，支持各种标准协议和功能，能够满足您的 DNS 基础架构需求。
