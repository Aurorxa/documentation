---
title: 如何处理内核恐慌（kernel panic）
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.4
tags:
  - kernel
  - kernel panic
  - rescue
---

## 简介

有时，内核安装会出错，您必须进行回退。

发生这种情况的原因很多，例如 `/boot` 分区空间不足、安装中断或第三方应用程序问题。

幸运的是，我们总是有办法挽救局面。

## 尝试使用之前的内核重启

首先要尝试的是使用之前的内核重启。

* 重启系统。
* 一旦到达 GRUB 2 启动屏幕，将选择移动到对应于之前内核的菜单条目，然后按 `enter` 键。

系统重启后，您可以修复它。

如果系统无法启动，请尝试**救援模式**（见下文）。

## 卸载损坏的内核

最简单的方法是卸载不起作用的内核版本，然后重新安装它。

!!! Note

    无法卸载您正在运行的内核。 
    
    要显示当前运行的内核版本： 
    
    ```bash
    uname -r
    ```

检查已安装内核的列表：

```bash
dnf list installed kernel\* | sort -V
```

但是下面的命令可能更实用，因为它只返回安装了多个版本的软件包：

```bash
dnf repoquery --installed --installonly
```

要删除特定内核，您可以使用 `dnf`，指定您之前获取的内核版本：

```bash
dnf remove kernel-core-<version>
```

示例：

```bash
dnf remove kernel-5.14.0-427.20.1.el9_4.x86_64
```

或使用 `dnf repoquery` 命令：

```bash
dnf remove $(dnf repoquery --installed --installonly --latest=1)
```

您现在可以升级您的系统并尝试重新安装最新的内核版本。

```bash
dnf update
```

重启并查看这次新内核是否正常工作。

## 救援模式

救援模式对应于旧的单用户模式。

!!! Note

    要进入救援模式，您必须提供 root 密码。

要进入救援模式，最简单的方法是在 grub 菜单中选择以 `0-rescue-*` 开头的行。

另一种方法是在 grub 菜单中编辑任何一行（按 'e' 键），在以 `linux` 开头的行末添加 `systemd.unit=rescue.target`，然后按 `ctrl+x` 将系统启动到救援模式。

!!! Note

    此时您处于 qwerty 模式。

一旦进入救援模式并输入 root 密码，您就可以修复系统了。

为此，您可能需要使用 `ip ad add ...` 配置临时 IP 地址（参见我们的管理员指南的网络章节）。

## 最后一招：Anaconda 救援模式

如果上述方法都不起作用，您仍然可以从安装 ISO 启动并修复系统。

本文档不涵盖此方法。

## 系统维护

### 清理旧内核版本

您可以删除旧的已安装内核软件包，仅保留最新版本和当前运行内核的版本：

```bash
dnf remove --oldinstallonly
```

### 限制已安装的内核版本数量

我们可以通过编辑 `/etc/yum.conf` 文件并设置 **installonly_limit** 变量来限制内核版本的数量：

```text
installonly_limit=3
```

!!! Note

    您应始终至少保留最新的内核版本和一个备份版本。
