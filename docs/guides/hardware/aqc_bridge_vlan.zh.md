---
title: 在 Marvell AQC 系列网卡上启用 VLAN 透传
author: Neel Chauhan
contributors: Steven Spencer
tested_with: 9.6
tags:
  - hardware
---

## 简介

作者在其家庭服务器中使用了一块基于 Marvell AQC107 的 NIC（网络接口卡），该服务器上运行着一个用于虚拟化防火墙的虚拟机。不幸的是，Rocky Linux 自带的 Marvell AQC 驱动会剥离桥接接口上的 VLAN。这种情况就发生在作者的 OPNsense 虚拟机上。不过，这个问题是可以修复的。

## 前置条件与假设

使用本操作流程的最低要求如下：

* 一台装有 Marvell AQC 系列 NIC 的 Rocky Linux 服务器
* 使用 NetworkManager 配置网络

## 禁用 VLAN 过滤

只需一条命令即可禁用 VLAN 过滤：

    nmcli con modify enp1s0 ethtool.feature-rx-vlan-filter off

将 `enp1s0` 替换为您基于 AQC 的 NIC 的名称。

最后，需要重启系统。
