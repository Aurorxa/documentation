---
title: dnf - swap 命令
author: wale soyinka
contributors:
date: 2023-01-24
tags:
  - cloud images
  - containers
  - dnf
  - dnf swap
  - vim
  - vim-minimal
  - allowerasing
  - coreutils-single
---


# 简介

为了使容器镜像和云镜像尽可能小，发行版维护者和打包者有时会发布精简版的流行软件包。与容器或云镜像捆绑的精简版软件包示例包括 **vim-minimal、curl-minimal、coreutils-single** 等。

尽管一些随附的软件包是精简版，但它们通常完全能满足大多数用例的需求。

当精简版软件包不够用时，您可以使用 `dnf swap` 命令快速将精简版软件包替换为常规软件包。

## 目标

这篇 Rocky Linux GEMstone 演示了如何使用 **dnf** 将捆绑的 `vim-minimal` 软件包替换（swap）为常规的 `vim` 软件包。

## 检查现有的 `vim` 变体

以具有管理权限的用户登录到容器或虚拟机环境后，首先验证已安装的 `vim` 软件包变体。输入：

```bash
# rpm -qa | grep  ^vim
vim-minimal-9.1.083-5.el10_0.1.x86_64
```

您的系统上安装了 `vim-minimal`。

## 将 `vim-minimal` 替换（swap）为 `vim`

使用 `dnf` 将已安装的 `vim-minimal` 软件包替换为常规的 `vim` 软件包。

```bash
# dnf -y swap vim-minimal vim
```

## 检查新的 `vim` 软件包变体

要确认更改，再次查询 rpm 数据库以获取已安装的 `vim` 软件包，运行：

```bash
# rpm -qa | grep  ^vim
vim-enhanced-9.1.083-5.el10_0.1.x86_64
```

大功告成！

## 备注

DNF Swap 命令

**语法**：

```bash
dnf [options] swap <package-to-be-removed> <replacement-package>
```

在底层，`dnf swap` 使用 DNF 的 `--allowerasing` 选项来解决任何软件包冲突问题。因此，本 GEMstone 中演示的 `vim-minimal` 示例也可以通过运行以下命令来实现：

```bash
dnf install -y --allowerasing vim
```
