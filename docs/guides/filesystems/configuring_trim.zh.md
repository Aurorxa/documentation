---
title: 配置 TRIM
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - trim
  - ssd
  - 存储
---

# 配置 TRIM

## 引言

TRIM 是一个命令，用于通知 SSD（固态硬盘）哪些数据块不再使用，从而可以被擦除以供将来使用。定期运行 TRIM 可以维持 SSD 的性能和使用寿命。在 Linux 中，您可以通过以下方式之一来配置 TRIM：

* 定期使用 `fstrim` 命令运行。
* 使用 `mount` 选项启用连续 TRIM（discard 选项）。

由于连续 TRIM 可能会影响某些系统上的性能，推荐使用定期 TRIM。本文档将向您展示如何启用它。

## 前提条件

* 一台运行 Rocky Linux 的服务器，配备 SSD 存储
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验

## 检查 TRIM 支持

在配置 TRIM 之前，请确保您的 SSD 和文件系统支持 TRIM。

```bash
lsblk --discard
```

如果 `DISC-GRAN` 和 `DISC-MAX` 列包含非零值，表示您的设备支持 TRIM。

检查文件系统挂载选项：

```bash
mount | grep discard
```

如果返回结果，说明 discard 选项已启用。

## 使用定期 TRIM

### 手动运行 TRIM

```bash
fstrim -v /
```

### 启用定期 TRIM

`fstrim.service` 是由 `util-linux` 软件包提供的 systemd 定时器。要启用它：

```bash
systemctl enable --now fstrim.timer
```

验证定时器是否已启用：

```bash
systemctl status fstrim.timer
```

默认情况下，定时器每周运行一次 `fstrim`。

## 使用连续 TRIM（不推荐）

如果您希望使用 discard 选项，在 `/etc/fstab` 中添加 `discard` 选项：

```text
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx / ext4 defaults,discard 0 1
```

编辑 `fstab` 后重新挂载或重启系统。

## 结论

TRIM 是 SSD 维护的核心功能，能够确保持久的性能。通过使用定期 TRIM 而非连续 TRIM，您可以在维持 SSD 健康的同时避免性能损失。`fstrim.timer` systemd 单元提供了一种简单自动化的方式来设置定期 TRIM。
