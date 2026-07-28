---
title: 2 ZFS 设置
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd zfs
---

# 第 2 章：ZFS 设置

在本章中，你需要以 root 用户身份操作，或者能够通过 `sudo` 成为 root。

如果你已经安装了 ZFS，本节将引导你完成 ZFS 的设置。

## 启用 ZFS 并设置存储池

首先，输入此命令：

```bash
/sbin/modprobe zfs
```

如果没有错误，它将返回提示符且不回显任何内容。如果你收到错误，请立即停止并开始排查故障。再次确认安全启动已关闭。这可能是最可能的原因。

接下来需要检查系统上的磁盘，找出操作系统位于何处，以及有哪些磁盘可用于 ZFS 存储池。使用 `lsblk` 来做到这一点：

```bash
lsblk
```

将返回类似以下内容（你的系统会有所不同！）：

```bash
AME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0    7:0    0  32.3M  1 loop /var/lib/snapd/snap/snapd/11588
loop1    7:1    0  55.5M  1 loop /var/lib/snapd/snap/core18/1997
loop2    7:2    0  68.8M  1 loop /var/lib/snapd/snap/lxd/20037
sda      8:0    0 119.2G  0 disk
├─sda1   8:1    0   600M  0 part /boot/efi
├─sda2   8:2    0     1G  0 part /boot
├─sda3   8:3    0  11.9G  0 part [SWAP]
├─sda4   8:4    0     2G  0 part /home
└─sda5   8:5    0 103.7G  0 part /
sdb      8:16   0 119.2G  0 disk
├─sdb1   8:17   0 119.2G  0 part
└─sdb9   8:25   0     8M  0 part
sdc      8:32   0 149.1G  0 disk
└─sdc1   8:33   0 149.1G  0 part
```

在此列表中，你可以看到 */dev/sda* 正被操作系统使用。你将使用 */dev/sdb* 作为 zpool。请注意，如果你有许多可用的硬盘，你可能需要考虑使用 raidz（一种专门为 ZFS 设计的软件 raid）。

这超出了本文档的范围，但绝对是生产环境中需要考虑的事项。它提供了更好的性能和冗余。现在，在你已识别的单个设备上创建存储池：

```bash
zpool create storage /dev/sdb
```

这表示在设备 */dev/sdb* 上创建一个名为 "storage" 的 ZFS 池。

创建存储池后，再次重启服务器。
