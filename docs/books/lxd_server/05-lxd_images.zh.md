---
title: 5 设置与管理镜像
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd images
---

# 第 5 章：设置与管理镜像

在本章中，你需要以非特权用户身份运行命令（如果从本手册开头开始，即为 "lxdadmin"）。

## 列出可用镜像

你可能迫不及待地想开始使用容器了。有许多容器操作系统可供选择。要了解有多少种可能，请输入此命令：

```bash
lxc image list images: | more
```

按空格键翻页浏览列表。这个容器和虚拟机列表在不断增长。

你最**不**想做的就是翻页寻找要安装的容器镜像，尤其是当你已经知道要创建什么镜像时。修改命令仅显示 Rocky Linux 安装选项：

```bash
lxc image list images: | grep rocky
```

这会得到一个更易于管理的列表：

```bash
| rockylinux/8 (3 more)                    | 0ed2f148f7c6 | yes    | Rockylinux 8 amd64 (20220805_02:06)          | x86_64       | CONTAINER       | 128.68MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/8 (3 more)                    | 6411a033fdf1 | yes    | Rockylinux 8 amd64 (20220805_02:06)          | x86_64       | VIRTUAL-MACHINE | 643.15MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/8/arm64 (1 more)              | e677777306cf | yes    | Rockylinux 8 arm64 (20220805_02:29)          | aarch64      | CONTAINER       | 124.06MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/8/cloud (1 more)              | 3d2fe303afd3 | yes    | Rockylinux 8 amd64 (20220805_02:06)          | x86_64       | CONTAINER       | 147.04MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/8/cloud (1 more)              | 7b37619bf333 | yes    | Rockylinux 8 amd64 (20220805_02:06)          | x86_64       | VIRTUAL-MACHINE | 659.58MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/8/cloud/arm64                 | 21c930b2ce7d | yes    | Rockylinux 8 arm64 (20220805_02:06)          | aarch64      | CONTAINER       | 143.17MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/9 (3 more)                    | 61b0171b7eca | yes    | Rockylinux 9 amd64 (20220805_02:07)          | x86_64       | VIRTUAL-MACHINE | 526.38MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/9 (3 more)                    | e7738a0e2923 | yes    | Rockylinux 9 amd64 (20220805_02:07)          | x86_64       | CONTAINER       | 107.80MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/9/arm64 (1 more)              | 917b92a54032 | yes    | Rockylinux 9 arm64 (20220805_02:06)          | aarch64      | CONTAINER       | 103.81MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/9/cloud (1 more)              | 16d3f18f2abb | yes    | Rockylinux 9 amd64 (20220805_02:06)          | x86_64       | CONTAINER       | 123.52MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/9/cloud (1 more)              | 605eadf1c512 | yes    | Rockylinux 9 amd64 (20220805_02:06)          | x86_64       | VIRTUAL-MACHINE | 547.39MB  | Aug 5, 2022 at 12:00am (UTC)  |
| rockylinux/9/cloud/arm64                 | db3ce70718e3 | yes    | Rockylinux 9 arm64 (20220805_02:06)          | aarch64      | CONTAINER       | 119.27MB  | Aug 5, 2022 at 12:00am (UTC)  |
```

## 安装、重命名和列出镜像

对于第一个容器，你将使用 "rockylinux/8"。要安装它，你*可以*使用：

```bash
lxc launch images:rockylinux/8 rockylinux-test-8
```

这将创建一个名为 "rockylinux-test-8" 的基于 Rocky Linux 的容器。你可以在创建后重命名容器，但首先需要停止容器，容器在创建后会自动启动。

要手动启动容器，使用：

```bash
lxc start rockylinux-test-8
```

要重命名镜像（这里不实际执行，但这是操作方法），首先停止容器：

```bash
lxc stop rockylinux-test-8
```

使用 `move` 命令更改容器名称：

```bash
lxc move rockylinux-test-8 rockylinux-8
```

如果你还是按此说明操作了，请停止容器并将其移回原始名称，以便继续跟随操作。

出于本指南的目的，现在再安装两个镜像：

```bash
lxc launch images:rockylinux/9 rockylinux-test-9
```

和

```bash
lxc launch images:ubuntu/22.04 ubuntu-test
```

通过列出镜像来检查你拥有的内容：

```bash
lxc list
```

将返回以下内容：

```bash
+-------------------+---------+----------------------+------+-----------+-----------+
|       NAME        |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-8 | RUNNING | 10.146.84.179 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-9 | RUNNING | 10.146.84.180 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| ubuntu-test       | RUNNING | 10.146.84.181 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
```
