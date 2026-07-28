---
title: XFS 恢复
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - xfs
  - 文件系统
  - 恢复
---

# XFS 文件系统恢复

## 引言

XFS 是一个高性能日志文件系统，由 SGI（Silicon Graphics, Inc.）开发，是 Red Hat Enterprise Linux 及其衍生版本（如 Rocky Linux）的默认文件系统。XFS 因其出色的可扩展性以及对大文件和大型目录树的高效处理而闻名。尽管 XFS 是一个健壮的文件系统，但在系统崩溃、意外断电或其他事件后，仍可能会出现不一致性。

XFS 的错误恢复通常由 `xfs_repair` 工具处理。该工具可以检查并修复 XFS 文件系统的不一致性。

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 熟悉 `vi`、`nano` 等命令行编辑器

## 检查文件系统

在尝试恢复文件系统之前，最好运行一个不带修复选项的 `xfs_repair` 检查，以评估文件系统上存在的问题。为此，卷**必须**以只读模式挂载或处于卸载状态。

```bash
xfs_repair -n /dev/sda1
```

如果文件系统是系统的根 (`/`) 文件系统，您需要在救援模式或 Live CD 环境中执行恢复操作，因为在挂载状态时运行 `xfs_repair` 可能会损坏文件系统。

## XFS 恢复工具

### `xfs_repair`

`xfs_repair` 是一个用于修复 XFS 文件系统的实用工具。它会扫描文件系统，查找不一致性，并尝试修复它们。当您的 XFS 文件系统在断电或系统崩溃后无法挂载时，这个工具非常有用。

```bash
xfs_repair /dev/sda1
```

如果不带任何选项运行 `xfs_repair`，它将首先以只读模式扫描卷，然后开始修复阶段。

### `xfs_check`

已被弃用。使用 `xfs_repair -n` 代替。

### `xfs_db`

`xfs_db` 是一个用于调试 XFS 文件系统的交互式工具。它可以检查文件系统结构、超级块、分配组元数据等。

用法示例：

```bash
xfs_db -c "freesp -s" /dev/sda1
```

### `xfs_admin`

`xfs_admin` 允许您更改 XFS 文件系统的参数，例如 UUID、标签等。

### `xfs_fsr`

`xfs_fsr` 是一个文件系统碎片整理工具。它通过重新组织文件系统中的数据来减少碎片。

### `xfs_metadump`

`xfs_metadump` 生成 XFS 文件系统元数据(metadata)的副本，并将其保存到文件中，以便后序分析。这对于诊断文件系统的问题非常有用，尤其是在无需发送整个文件系统拷贝的情况下向他人发送信息。

### `xfs_growfs`

`xfs_growfs` 用于增大（扩展）XFS 文件系统的大小。

## 从单用户模式恢复

如果系统无法正常启动，您可以尝试以单用户模式启动，然后运行 `xfs_repair`。

在启动时，按 `e` 进入 GRUB 编辑器。找到以 `linux` 或 `linux16` 开头的行，并在行尾添加 `single` 以进入单用户模式。如果用户模式下无法访问 XFS 工具，您可能需要使用安装介质的救援模式。

## 安装介质救援模式

如果 `xfs_repair` 无法运行，您可以从 Rocky Linux 安装介质引导，并进入救援模式。选择 "Troubleshooting" -> "Rescue a Rocky Linux system"。然后进入 shell。

在 shell 中，识别要修复的文件系统，运行：

```bash
xfs_repair /dev/sda1
```

修复完成后，重新启动：

```bash
reboot
```

## 常见问题

### 日志区域(log area)脏数据

如果 `xfs_repair` 输出指出存在脏日志(dirty log)：

```bash
xfs_repair -L /dev/sda1
```

!!! warning

    此选项会清除日志，这可能会导致最近的数据丢失。仅在所有其他方法都失败时使用。

## 备份的重要性

无论文件系统的健壮性如何，保证数据安全的最佳方法是定期备份。虽然 `xfs_repair` 可以修复许多文件系统不一致性，但它不能防止硬件故障或用户错误导致的数据丢失。

在 Rocky Linux 和其他 Linux 发行版中，有一些现成的备份工具，包括 `rsync`、`tar`、`dump`、`Amanda` 和 `Bacula`。

## 结论

XFS 提供了一套强大的恢复工具。了解 `xfs_repair`、`xfs_db` 和其他 XFS 专用工具是管理 Rocky Linux 系统的重要组成部分。对关键数据始终保留备份，不要等到灾难发生时才想起备份。
