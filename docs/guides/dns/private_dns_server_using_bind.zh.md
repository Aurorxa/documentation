---
title: 使用 Bind 搭建私有 DNS 服务器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - dns
  - bind
---

# 私有 DNS 服务器

## 引言

大多数计算机网络都依赖于 DNS 协议，通过名称来识别和检索公共或私有网络上的计算机和设备。公共 DNS 服务可以通过您的 ISP (Internet Service Provider)、外部解析器(External Resolver, 如 Google DNS)来配置，但通常不会用于私有网络中的资源。这就是私有 DNS 服务器的用武之地。

BIND (Berkeley Internet Name Domain) 是一款功能齐全、被广泛使用的 DNS 软件。它提供了一个强大的组件组合，允许您创建权威 DNS 服务器(authoritative DNS server)、递归解析器(recursive resolver)以及转发器。本指南将专注于使用 BIND 搭建私有 DNS 服务器，并支持正向和反向解析。

本指南中，我们将为一个虚构的私有网络配置 DNS 服务器。主要 DNS 服务器位于 `ns1.example.com`，支持 192.168.0.0/24 C 类私有网络中的电子邮件和 Web 服务。

## 前提条件

* 一台运行 Rocky Linux 的服务器（主要 DNS 服务器 `ns1.example.com`）
* 一台能访问 DNS 的客户端
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验
* 对 DNS 有基本了解

## 安装 BIND

```bash
dnf install bind bind-utils
```

## 配置服务器

`bind` 软件包会将默认配置文件安装在 `/etc/named.conf` 中。备份文件。

```bash
cp /etc/named.conf /etc/named.conf.orig
```

区域数据(zone data)将存储在 `/var/named/` 目录中，因为这是 `named` 服务默认的工作目录。对于本指南，我们使用默认位置。

### 主配置文件

编辑 `/etc/named.conf` 文件。我们将配置 BIND 为仅监听本地网络，并定义所需的 ACL（访问控制列表）、区域和日志记录。

```text
acl "trusted" {
        192.168.0.0/24;
        127.0.0.1;
};

options {
        listen-on port 53 { 127.0.0.1; 192.168.0.10; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { trusted; };
        allow-recursion { trusted; };
        recursion yes;
        dnssec-validation yes;
        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
};
```

* `listen-on` 限制 BIND 只监听本地回环地址和指定的私有 IP 地址。
* `allow-query` 和 `allow-recursion` 限制查询和递归操作仅在 `acl trusted` 中定义的可信网络中。
* `recursion yes` 启用递归查询。
* `forwarders` 将所有非权威查询转发到您定义的外部解析器(External Resolvers)。

在文件底部，添加区域定义：

```text
zone "example.com" {
    type master;
    file "example.com.zone";
    allow-transfer { none; };
};

zone "0.168.192.in-addr.arpa" {
    type master;
    file "0.168.192.in-addr.arpa.zone";
    allow-transfer { none; };
};
```

### 区域文件

现在，创建正向查找区域文件 `/var/named/example.com.zone`：

```text
$TTL 86400
@       IN SOA  ns1.example.com. admin.example.com. (
                2024101401 ; serial
                3600       ; refresh
                1800       ; retry
                604800     ; expire
                86400      ; minimum TTL
                )
              IN NS         ns1.example.com.
              IN MX 10      mail.example.com.
              IN A          192.168.0.13
ns1           IN A          192.168.0.10
mail          IN A          192.168.0.13
www           IN A          192.168.0.14
```

接下来，反向查找区域文件 `/var/named/0.168.192.in-addr.arpa.zone`：

```text
$TTL 86400
@       IN SOA  ns1.example.com. admin.example.com. (
                2024101401 ; serial
                3600       ; refresh
                1800       ; retry
                604800     ; expire
                86400      ; minimum TTL
                )
              IN NS         ns1.example.com.
10            IN PTR        ns1.example.com.
13            IN PTR        mail.example.com.
14            IN PTR        www.example.com.
```

确保区域文件的权限正确：

```bash
chown root:named /var/named/example.com.zone /var/named/0.168.192.in-addr.arpa.zone
chmod 640 /var/named/example.com.zone /var/named/0.168.192.in-addr.arpa.zone
```

### 启动并启用 BIND 服务

```bash
systemctl enable --now named
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

### 检查 `named` 运行状态

```bash
systemctl status named
```

### 检查区域

使用 `named-checkzone` 测试区域文件：

```bash
named-checkzone example.com /var/named/example.com.zone
named-checkzone 0.168.192.in-addr.arpa /var/named/0.168.192.in-addr.arpa.zone
```

### 使用 `dig` 测试查询

```bash
dig @192.168.0.10 mail.example.com
dig @192.168.0.10 -x 192.168.0.13
```

## 客户端配置

为了让客户端使用您的 DNS 服务器，您需要配置客户端以使用新服务器。最简单的方法是直接修改 `/etc/resolv.conf`，但请注意 NetworkManager 可能会覆盖此文件。一种更好的方法是通过 `nmcli` 进行配置：

```bash
nmcli con mod "your connection name" ipv4.dns "192.168.0.10"
nmcli con down "your connection name" && nmcli con up "your connection name"
```

## 结论

您现在拥有一个功能完备的私有 DNS 服务器，它使用 BIND 提供正反向解析服务。该私有 DNS 服务器可以为您的私有网络提供名称解析服务，提高生产效率并简化资源管理。
