---
title: 使用 Xorriso 写入实体 CD/DVD
author: Joseph Brinkman 
contributors: Steven Spencer, Ganna Zhyrnova
---

## 简介

我最近发现在 Rocky Linux 上使用图形化工具将混合 ISO 烧录到实体 CD/DVD 非常困难。幸运的是，Xorriso 是一个简单易用的 CLI 应用程序，可以很好地处理此任务！

## 问题描述

将 ISO 烧录到实体 CD/DVD。

## 前置条件

- 互联网连接
- 熟悉终端操作
- CD/DVD RW 驱动器

## 操作步骤

**安装 Xorriso**：

   ```bash
   sudo dnf install xorriso -y
   ```

**将 ISO 写入光盘**：

   ```bash
   sudo xorriso -as cdrecord -v dev=/dev/sr0 -blank=as_needed -dao Rocky-10.1-x86_64-boot.iso -eject
   ```

## 附加信息

Xorriso 依赖于 C 库 `libisofs`。您可以在 [Fedora 的包监控页面](https://packages.fedoraproject.org/pkgs/libisofs/libisofs/index.html) 了解更多关于 `libisofs` 的信息。

## 总结

在这篇 gemstone 中，您学习了如何使用 Xorriso 将 ISO 写入实体光盘。请记住，Xorriso 还可以用于将其他文件类型写入实体光盘，但我发现它对于图形化工具无法处理的混合 ISO 格式特别方便。
