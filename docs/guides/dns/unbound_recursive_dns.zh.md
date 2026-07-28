---
title: Unbound 递归 DNS 服务器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - dns
  - unbound
---

# 递归 DNS 服务器 —— Unbound

## 引言

Unbound 是由 NLnet Labs 开发的一款递归验证 DNS 解析器(recursive validating DNS resolver)，采用 C 语言编写，使用 BSD 许可证分发。它提供高性能的 DNS 解析和缓存服务，同时还支持 DNSSEC 验证。Unbound 的核心特性包括：

* 递归解析(recursive resolving)：不依赖其他服务器，通过从根服务器(root server)开始，按层级依次查询各级服务器，并缓存查询结果，自行完成 DNS 解析。
* 内置 DNSSEC 验证：提供高安全性，防御 DNS 欺骗和缓存投毒攻击。
* 支持现代协议和功能，如 DNS over TLS (DoT) 和 DNS over HTTPS (DoH)。
* 有助于保护用户的隐私。

本指南将向您展示如何在 Rocky Linux 上安装和配置一个递归 DNS 服务器。

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验
* 对 DNS 有基本了解

## 安装 Unbound

```bash
dnf install unbound
```

## 配置服务器

安装完成后，替换默认配置文件。备份原始文件：

```bash
cp /etc/unbound/unbound.conf /etc/unbound/unbound.conf.orig
```

编辑配置文件 `/etc/unbound/unbound.conf`：

```text
server:
    interface: 0.0.0.0
    port: 53
    access-control: 0.0.0.0/0 refuse
    access-control: 127.0.0.0/8 allow
    access-control: 192.168.0.0/24 allow
    verbosity: 1
    do-ip4: yes
    do-ip6: yes
    do-udp: yes
    do-tcp: yes
    prefer-ip6: no
    hide-identity: yes
    hide-version: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: yes
    cache-min-ttl: 3600
    cache-max-ttl: 86400
    infra-host-ttl: 60
    infra-lame-ttl: 120
    edns-buffer-size: 1472
    msg-buffer-size: 65552
    num-queries-per-thread: 4096
    outgoing-range: 8192
    num-threads: 4
    msg-cache-size: 256m
    rrset-cache-size: 512m
    key-cache-size: 256m
    neg-cache-size: 256m
    prefetch: yes
    prefetch-key: yes
    minimal-responses: yes
    rrset-roundrobin: yes
    qname-minimisation: yes
    serve-expired: yes
    val-clean-additional: yes
    val-permissive-mode: no
    val-log-level: 2
    trust-anchor-file: "/var/lib/unbound/root.key"
    auto-trust-anchor-file: "/var/lib/unbound/root.key"
```

* `access-control` 控制哪些网络可以查询您的服务器。默认情况下，拒绝所有访问，然后允许本地回环和私有网络。
* 启用 `hide-identity` 和 `hide-version` 隐藏服务器信息，增强安全性。
* 各种缓存大小和预取选项可提高性能和查询速度。
* `qname-minimisation` 通过最小化查询名称信息来提升隐私性。

### 初始化并更新 DNSSEC 根信任锚

```bash
sudo -u unbound unbound-anchor -a /var/lib/unbound/root.key
unbound-checkconf
```

### 启动并启用 Unbound 服务

```bash
systemctl enable --now unbound
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

### 检查 `unbound` 运行状态

```bash
systemctl status unbound
```

### 使用 `dig` 测试查询

```bash
dig @127.0.0.1 example.com
dig @127.0.0.1 rockylinux.org
```

您还可以测试 DNSSEC 验证功能：

```bash
dig @127.0.0.1 sigfail.verteiltesysteme.net
dig @127.0.0.1 sigok.verteiltesysteme.net
```

## 客户端配置

为了让客户端使用您的 DNS 服务器，您需要配置客户端以使用新服务器。最简单的方法是直接修改 `/etc/resolv.conf`，但请注意 NetworkManager 可能会覆盖此文件。一种更好的方法是通过 `nmcli` 进行配置：

```bash
nmcli con mod "your connection name" ipv4.dns "192.168.0.10"
nmcli con down "your connection name" && nmcli con up "your connection name"
```

## 结论

您现在拥有一个功能完备的递归 DNS 服务器，它使用 Unbound 提供高性能、安全的递归 DNS 解析服务。Unbound 提供隐私功能，支持 DNSSEC，并可作为您网络中的完整解析解决方案，其高性能特性可在大型网络中提供高效的 DNS 服务。
