---
title: 重新生成 `initramfs`
author: Neel Chauhan
contributors: Steven Spencer
tested_with: 9.4
tags:
  - hardware
---

## 简介

`initramfs` 是 Linux 内核内部的根文件系统，用于帮助引导系统。它包含引导 Linux 所需的核心模块。

有时，Linux 管理员可能希望重新生成 `initramfs`，例如想要将某个驱动加入黑名单或包含一个带外模块。作者曾这样操作来[在 Minisforum MS-01 上启用 Intel vPro](https://spaceterran.com/posts/step-by-step-guide-enabling-intel-vpro-on-your-minisforum-ms-01-bios/)。

## 要求

使用本操作流程的最低要求如下：

* 一台 Rocky Linux 系统或虚拟机（非容器）
* 对内核设置的更改，例如将模块加入黑名单或添加模块

## 重新生成 `initramfs`

要重新生成 `initramfs`，应首先备份现有的 `initramfs`：

```bash
cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-$(date +%m-%d-%H%M%S).img
```

接下来，运行 `dracut` 重新生成 `initramfs`：

```bash
dracut -f /boot/initramfs-$(uname -r).img $(uname -r)
```

然后重启：

```bash
reboot
```

## 总结

Linux 内核功能强大且高度模块化。一些用户可能希望允许或禁止某些模块，重新生成 `initramfs` 就可以实现这一点。
