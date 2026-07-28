---
title: 7 容器配置选项
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd configuration
---

# 第 7 章：容器配置选项

在本章中，你需要以非特权用户身份运行命令（如果从本手册开头开始，即为 "lxdadmin"）。

安装后配置容器的选项非常丰富。然而，在了解这些之前，让我们先检查一下容器的 `info` 命令。在此示例中，你将使用 ubuntu-test 容器：

```bash
lxc info ubuntu-test
```

将显示以下内容：

```bash
Name: ubuntu-test
Location: none
Remote: unix://
Architecture: x86_64
Created: 2021/04/26 15:14 UTC
Status: Running
Type: container
Profiles: default, macvlan
Pid: 584710
Ips:
  eth0:    inet    192.168.1.201    enp3s0
  eth0:    inet6    fe80::216:3eff:fe10:6d6d    enp3s0
  lo:    inet    127.0.0.1
  lo:    inet6    ::1
Resources:
  Processes: 13
  Disk usage:
    root: 85.30MB
  CPU usage:
    CPU usage (in seconds): 1
  Memory usage:
    Memory (current): 99.16MB
    Memory (peak): 110.90MB
  Network usage:
    eth0:
      Bytes received: 53.56kB
      Bytes sent: 2.66kB
      Packets received: 876
      Packets sent: 36
    lo:
      Bytes received: 0B
      Bytes sent: 0B
      Packets received: 0
      Packets sent: 0
```

这里面有很多有用的信息，从应用的配置文件到使用的内存、使用的磁盘空间等等。

## 关于配置和一些选项

默认情况下，LXD 将为容器分配所需的系统内存、磁盘空间、CPU 核心和其他资源。但如果想要更精细地控制呢？这完全是可以做到的。

然而，这样做有利有弊。例如，如果你分配了系统内存而容器没有全部使用，你就阻止了其他可能实际需要它的容器使用。反过来也可能发生。如果一个容器想要使用超过其内存份额，它可能会阻止其他容器获得足够的内存，从而影响它们的性能。

只需记住，配置容器的每个操作**都可能**在其他地方产生负面影响。

与其逐一介绍所有配置选项，不如使用 tab 自动补全来查看可用选项：

```bash
lxc config set ubuntu-test
```

然后按 ++tab++。

这将显示所有容器配置选项。如果你对某个配置选项的功能有疑问，请前往 [LXD 官方文档](https://documentation.ubuntu.com/lxd/en/latest/config-options/)并搜索配置参数，或者在 Google 中搜索整个字符串，如 `lxc config set limits.memory`，然后查看搜索结果。

这里我们查看一些最常用的配置选项。例如，如果你想设置容器可以使用的最大内存量：

```bash
lxc config set ubuntu-test limits.memory 2GB
```

这表示如果有 2GB 内存可用，容器实际上可以使用超过 2GB。这是一个软限制。

```bash
lxc config set ubuntu-test limits.memory.enforce 2GB
```

这表示容器永远不能使用超过 2GB 的内存，无论当前是否可用。在这种情况下是硬限制。

```bash
lxc config set ubuntu-test limits.cpu 2
```

这表示将容器可以使用的 CPU 核心数限制为 2。

!!! note

    当本文档针对 Rocky Linux 9.0 重写时，9.x 的 ZFS 仓库尚不可用。因此，我们所有的测试容器都是在初始化过程中使用 "dir" 构建的。这就是为什么下面的示例显示 "dir" 而不是 "zfs" 存储池。

还记得在 ZFS 章节中设置存储池时的操作吗？你将池命名为 "storage"，但你完全可以给它起任何名字。如果你想要检查它，可以使用以下命令，该命令对其他池类型也同样有效（以 dir 为例）：

```bash
lxc storage show storage
```

显示以下内容：

```bash
config:
  source: /var/snap/lxd/common/lxd/storage-pools/storage
description: ""
name: storage
driver: dir
used_by:
- /1.0/instances/rockylinux-test-8
- /1.0/instances/rockylinux-test-9
- /1.0/instances/ubuntu-test
- /1.0/profiles/default
status: Created
locations:
- none
```

这表明我们所有的容器都使用 dir 存储池。当使用 ZFS 时，你还可以对容器设置磁盘配额。以下是为 ubuntu-test 容器设置 2GB 磁盘配额的命令：

```bash
lxc config device override ubuntu-test root size=2GB
```

如前所述，除非容器想要使用远超其资源份额，否则应谨慎使用配置选项。LXD 在很大程度上会自行良好地管理环境。

还有许多其他选项可能对某些人感兴趣。自行研究这些选项将帮助你在你的环境中确定其中哪些有价值。
