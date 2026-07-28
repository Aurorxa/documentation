---
title: 7 容器配置选项
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with:  9.4
tags:
  - incus 
  - enterprise
  - incus configuration
---

在本章的全部内容中，你需要以你的非特权用户身份运行命令（如果你从一开始就一直在跟随本手册，则为 "incusadmin"）。

安装后有大量用于配置容器的选项。然而，在查看这些选项之前，让我们先检查容器的 `info` 命令。在此示例中，你将使用 ubuntu-test 容器：

```bash
incus info ubuntu-test
```

这将显示以下内容：

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

这里有来自应用的配置文件、内存、正在使用的磁盘空间等很好的信息。

## 关于配置和一些选项的说明

默认情况下，Incus 会为容器分配所需的系统内存、磁盘空间、CPU 核心和其他资源。但如果你想更具体地指定呢？这是可能的。

这样做有一些权衡。例如，如果你分配了系统内存但容器并未全部使用，你就阻止了另一个可能需要它的容器使用这些内存。反过来也可能发生。如果一个容器想要使用超过其份额的内存，它可能会阻止其他容器获取足够的内存，从而影响它们的性能。

请记住，你为配置容器所做的每一个操作都*可能*在别处产生不利影响。

与其逐一介绍所有配置选项，不如使用 tab 自动补全来查看可用的选项：

```bash
incus config set ubuntu-test
```

然后按 ++tab++。

这将显示配置容器的所有选项。如果你对某个配置选项的作用有疑问，请前往 [Incus 官方文档](https://linuxcontainers.org/incus/docs/main/config-options/)并搜索该配置参数，或 Google 整个字符串，如 `incus config set limits.memory`，并检查搜索结果。

这里，我们检查一些最常用的配置选项。例如，如果你想设置容器可以使用的最大内存量：

```bash
incus config set ubuntu-test limits.memory 2GB
```

这表示如果有可用内存，例如 2 GB，那么容器实际上可以使用超过 2GB。这是一个软限制。

```bash
incus config set ubuntu-test limits.memory.enforce 2GB
```

这表示容器永远不能使用超过 2GB 的内存，无论当前是否可用。这是一个硬限制。

```bash
incus config set ubuntu-test limits.cpu 2
```

这表示将容器可以使用的 CPU 核心数限制为 2。

!!! note

    当本文档为 Rocky Linux 9.0 重写时，用于 9 的 ZFS 仓库尚不可用。因此，我们所有的测试容器都是在 init 过程中使用 "dir" 构建的。这就是为什么下面的示例显示的是 "dir" 存储池而不是 "zfs" 存储池。

还记得你在 ZFS 章节中设置存储池的时候吗？你将池命名为 "storage"，但你可以将其命名为任何名称。如果你想检查这一点，可以使用以下命令，该命令同样适用于任何其他池类型（如 dir 所示）：

```bash
incus storage show storage
```

这将显示以下内容：

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

这表明我们所有的容器都使用我们的 dir 存储池。当使用 ZFS 时，你还可以在容器上设置磁盘配额。以下是对 ubuntu-test 容器设置 2GB 磁盘配额的命令：

```bash
incus config device override ubuntu-test root size=2GB
```

正如我之前所说，除非你有一个想要使用超过其份额的资源的容器，否则可以谨慎使用配置选项。在大多数情况下，Incus 将很好地管理环境。

还有许多其他选项可能对某些人感兴趣。做你自己的研究将帮助你确定其中哪些在你的环境中有价值。
