---
title: 5 镜像的设置与管理
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 9.4
tags:
  - incus 
  - enterprise
  - incus images
---

在本章的全部内容中，你需要以你的非特权用户身份运行命令（如果你从一开始就遵循了本手册，则为 "incusadmin"）。

## 列出可用的镜像

你可能迫不及待地想要开始使用容器。有许多容器操作系统的可能性。要了解有多少种可能性，请输入此命令：

```bash
incus image list images: | more
```

按空格键翻页浏览列表。这个容器和虚拟机的列表还在持续增长。

你**绝对不想**做的是去大海捞针般寻找要安装的容器镜像，特别是如果你知道自己要创建什么镜像的时候。修改命令，仅显示 Rocky Linux 安装选项：

```bash
incus image list images: | grep rocky
```

这会生成一个更易于管理的列表：

```bash
| rockylinux/8 (3 more)                    | dede6169bb45 | yes    | Rockylinux 8 amd64 (20240903_05:18)        | x86_64       | VIRTUAL-MACHINE | 850.75MiB  | 2024/09/02 19:00 CDT |
| rockylinux/8/arm64 (1 more)              | b749bad83e60 | yes    | Rockylinux 8 arm64 (20240903_04:40)        | aarch64      | CONTAINER       | 125.51MiB  | 2024/09/02 19:00 CDT |
| rockylinux/8/cloud (1 more)              | 4fefd464d25d | yes    | Rockylinux 8 amd64 (20240903_05:18)        | x86_64       | VIRTUAL-MACHINE | 869.95MiB  | 2024/09/02 19:00 CDT |
| rockylinux/8/cloud (1 more)              | 729891475172 | yes    | Rockylinux 8 amd64 (20240903_05:18)        | x86_64       | CONTAINER       | 148.81MiB  | 2024/09/02 19:00 CDT |
| rockylinux/8/cloud/arm64                 | 3642ec9652fc | yes    | Rockylinux 8 arm64 (20240903_04:52)        | aarch64      | CONTAINER       | 144.84MiB  | 2024/09/02 19:00 CDT |
| rockylinux/9 (3 more)                    | 9e5e4469e660 | yes    | Rockylinux 9 amd64 (20240903_03:29)        | x86_64       | VIRTUAL-MACHINE | 728.60MiB  | 2024/09/02 19:00 CDT |
| rockylinux/9 (3 more)                    | fff1706d5834 | yes    | Rockylinux 9 amd64 (20240903_03:29)        | x86_64       | CONTAINER       | 111.25MiB  | 2024/09/02 19:00 CDT |
| rockylinux/9/arm64 (1 more)              | d3a44df90d69 | yes    | Rockylinux 9 arm64 (20240903_04:49)        | aarch64      | CONTAINER       | 107.18MiB  | 2024/09/02 19:00 CDT |
| rockylinux/9/cloud (1 more)              | 4329a67099ba | yes    | Rockylinux 9 amd64 (20240903_03:28)        | x86_64       | VIRTUAL-MACHINE | 749.29MiB  | 2024/09/02 19:00 CDT |
| rockylinux/9/cloud (1 more)              | bc30d585b9f0 | yes    | Rockylinux 9 amd64 (20240903_03:28)        | x86_64       | CONTAINER       | 127.16MiB  | 2024/09/02 19:00 CDT |
| rockylinux/9/cloud/arm64                 | 5c38ddd506bd | yes    | Rockylinux 9 arm64 (20240903_04:38)        | aarch64      | CONTAINER       | 122.87MiB  | 2024/09/02 19:00 CDT |
```

## 安装、重命名和列出镜像

对于第一个容器，你将使用 "rockylinux/8"。要安装它，你可以使用：

```bash
incus launch images:rockylinux/8 rockylinux-test-8
```

这将创建一个名为 "rockylinux-test-8" 的基于 Rocky Linux 的容器。你可以在创建后重命名容器，但你需要先停止该容器，容器在创建时会自动启动。

要手动启动容器，使用：

```bash
incus start rockylinux-test-8
```

要重命名镜像（我们这里不这样做，但这是操作方法），首先停止容器：

```bash
incus stop rockylinux-test-8
```

使用 `move` 命令更改容器的名称：

```bash
incus move rockylinux-test-8 rockylinux-8
```

如果你无论如何都按照此说明操作了，请停止容器并将其移回原始名称以继续跟随本指南。

对于本指南，现在继续安装另外两个镜像：

```bash
incus launch images:rockylinux/9 rockylinux-test-9
```

和

```bash
incus launch images:ubuntu/22.04 ubuntu-test
```

通过列出你的镜像来检查你所拥有的内容：

```bash
incus list
```

这将返回：

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
