---
title: 在 AOOSTAR WTR PRO 上安装 Rocky Linux 10
author: Neel Chauhan
contributors: Steven Spencer
tested_with: 10.0
tags:
  - hardware
---

## 简介

AOOSTAR WTR PRO 是一款低功耗 x86 NAS（网络附加存储），配备四个硬盘托架。它是 HPE ProLiant MicroServer 更快、更便宜的替代方案。例如，作者购买了一台作为个人 NAS 使用。

虽然 WTR PRO 的设计目标是运行标准 Linux 发行版，但 Rocky Linux 安装程序无法在该设备上开箱即用。不过，仍然可以安装 Rocky Linux。

## 前置条件与假设

使用本操作流程的最低要求如下：

* 一个 Rocky Linux 安装程序 USB

* 一台 AOOSTAR WTR PRO 系统

## 引导 Rocky Linux 安装程序

首先，从 USB 引导。

如果 SSD 上已有现有操作系统，在启动 WTR PRO 时按 `Delete` 键。导航到 **Save & Exit** 并选择 USB。

当从 USB 引导进入 GRUB 菜单时，选择 **Troubleshooting**：

![GRUB 主菜单](../images/aoostar_1.png)

接着，选择 **Install Rocky Linux *VERSION* in basic graphics mode**：

![GRUB 故障排除菜单](../images/aoostar_2.png)

Rocky Linux 现在应该可以正常引导并安装了。

请注意，在安装 Rocky Linux 时不需要特殊的内核选项。
