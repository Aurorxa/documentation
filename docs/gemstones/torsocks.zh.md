---
title: torsocks - 通过 Tor/SOCKS5 路由流量
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
date: 2024-02-25
---

## `torsocks` 简介

`torsocks` 是一个将命令行应用程序的 IP 流量通过 [Tor](https://www.torproject.org/) 网络或 SOCKS5 服务器重新路由的工具。

## 使用 `torsocks`

```bash
dnf -y install epel-release
dnf -y install tor torsocks
systemctl enable --now tor
```

`torsocks` 命令的常用选项如下，正常情况下不需要额外的配置。选项放在要运行的应用程序之前（例如 `curl`）：

|选项|描述|
|---|---|
|--shell |创建一个带有 LD\_PRELOAD 的新 shell|
|-u USER |设置 SOCKS5 用户名|
|-p PASS |设置 SOCKS5 密码|
|-a ADDRESS |设置 SOCKS5 服务器地址|
|-P PORT |设置 SOCKS5 服务器端口|
|-i |启用 Tor 隔离|

通过 `torsocks` 使用 IP 检测器 [icanhazip.com](https://icanhazip.com/) 的示例（已脱敏）输出：

![torsocks output](./images/torsocks.png)

请注意，`torsocks` 的 IP 地址与直接 `curl` 的 IP 地址不同。
