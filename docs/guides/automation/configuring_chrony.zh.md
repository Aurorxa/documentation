---
title: 配置 chrony
author: Howard Van Der Wal
contributors: Steven Spencer, Ganna Zhyrnova
tested with: 8, 9, 10
ai_contributors: Claude (claude-opus-4-6)
tags:
- chrony
- ntp
- synchronization
- time
---

**知识水平**: :star: :star:
**阅读时间**: 20 分钟

## AI 使用声明

本文档遵循[此处的 AI 贡献政策](../contribute/ai-contribution-policy.md)。如发现说明中有任何错误，请告知我们。

## 简介

精确的时间同步是现代 Linux 系统运行的基础。Kerberos 认证、TLS 证书验证、分布式数据库、日志关联和集群调度器等服务都依赖于准确的系统时钟。在 Rocky Linux 上，`chrony` 是默认的 NTP 实现，取代了传统的 `ntpd` 守护进程。

`chrony`^1^ 专为网络连接不稳定或系统频繁休眠和恢复的环境设计。在稳定的网络条件下，`chrony` 可以达到约 35 微秒的精度，而 `ntpd` 为 234 微秒^1^。它还支持硬件时间戳和网络安全（NTS），这是 `ntpd` 所不具备的。

本指南涵盖在 Rocky Linux 上配置 `chrony` 的核心内容，包括选择 NTP 源、诊断同步问题、启用硬件时间戳、将 `chrony` 配置为本地 NTP 服务器以及处理离线网络。

## 前提条件

- 一台具有 `root` 或 `sudo` 访问权限的 Rocky Linux 8、9 或 10 系统。
- 已安装 `chrony` 软件包（Rocky Linux 默认安装）。
- 至少一个 NTP 服务器的网络访问权限（适用于联网系统）。

验证 `chrony` 是否已安装：

```bash
rpm -q chrony
```

## 默认 chrony 配置

Rocky Linux 自带一个默认的 `/etc/chrony.conf`，提供了开箱即用的时间同步设置。以下是 Rocky Linux 10 上的 RPM 默认配置：

（后续内容见完整翻译文件...）
