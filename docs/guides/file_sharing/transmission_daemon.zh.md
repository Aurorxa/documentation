---
title: Transmission BitTorrent 种子盒子
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - transmission
  - bittorrent
  - p2p
---

# Transmission BitTorrent 种子盒子

## 引言

Transmission 是一个轻量级的 BitTorrent 客户端，提供了一个简单易用的界面来下载和共享文件。它既可以作为独立应用使用，也可以作为无头(headless)守护进程运行在服务器上，并通过 Web 界面进行远程控制。

Transmission 的功能包括：

* 轻量且快速的 BitTorrent 引擎
* Web 界面用于远程管理
* 支持加密、PeX（Peer Exchange）、DHT（Distributed Hash Table）等现代 BitTorrent 特性
* 可以作为一个无头守护进程运行
* 计划下载和速度限制
* 支持种子优先级排序

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验

## 安装

```bash
dnf install transmission-daemon
```

## 配置 Transmission

启动 Transmission 会创建一个初始配置文件。先启动它：

```bash
systemctl start transmission-daemon
```

然后停止它，以便后续编辑配置文件：

```bash
systemctl stop transmission-daemon
```

现在编辑配置文件 `/var/lib/transmission/.config/transmission-daemon/settings.json`：

```json
{
    "rpc-authentication-required": true,
    "rpc-enabled": true,
    "rpc-password": "your-password",
    "rpc-port": 9091,
    "rpc-username": "your-username",
    "rpc-whitelist": "192.168.*.*",
    "download-dir": "/var/lib/transmission/Downloads",
    "incomplete-dir": "/var/lib/transmission/Downloads",
    "speed-limit-down": 1000,
    "speed-limit-up": 100
}
```

之后启动并启用服务：

```bash
systemctl enable --now transmission-daemon
```

您需要为下载目录设置适当的权限：

```bash
chown -R transmission:transmission /var/lib/transmission/Downloads
```

## 配置防火墙

允许访问 Web 界面：

```bash
firewall-cmd --add-port=9091/tcp --permanent
firewall-cmd --reload
```

## 访问 Web 界面

Transmission 的 Web 界面运行在 `http://server_ip:9091`。输入配置的用户名和密码进行身份验证。

![Transmission Web 界面](../images/transmission_web_interface.png)

## 结论

Transmission 是一个轻量级的 BitTorrent 种子盒子解决方案，能够在低资源的服务器上高效运行。其 Web 界面让远程管理变得非常方便。
