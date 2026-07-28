---
title: 8 容器快照
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd snapshots
---

# 第 8 章：容器快照

在本章中，你需要以非特权用户身份运行命令（如果从本手册开头开始，即为 "lxdadmin"）。

容器快照，以及快照服务器（稍后会详细介绍），可能是运行生产 LXD 服务器最重要的方面。快照确保快速恢复。在更新特定容器上运行的主要软件时，使用快照作为故障安全措施是一个好主意。如果更新过程中发生某些事情导致应用程序崩溃，只需恢复快照，你就可以在几秒钟内恢复运行。

作者曾在面向公众的 PowerDNS 服务器上使用 LXD 容器，多亏了在每次更新前拍摄快照，更新这些应用程序的过程变得不那么令人担忧。

你甚至可以在容器运行时对其进行快照。

## 快照过程

首先使用以下命令获取 ubuntu-test 容器的快照：

```bash
lxc snapshot ubuntu-test ubuntu-test-1
```

这里你将快照命名为 "ubuntu-test-1"，但你可以给它起任何名字。要确认你已获取快照，对容器执行 `lxc info`：

```bash
lxc info ubuntu-test
```

你已经查看过信息界面。如果滚动到底部，你现在会看到：

```bash
Snapshots:
  ubuntu-test-1 (taken at 2021/04/29 15:57 UTC) (stateless)
```

成功！我们的快照已就位。

进入 ubuntu-test 容器：

```bash
lxc exec ubuntu-test bash
```

使用 _touch_ 命令创建一个空文件：

```bash
touch this_file.txt
```

退出容器。

在将容器恢复到创建文件之前的状态时，恢复容器最安全的方式（尤其是在有许多更改的情况下）是先停止它：

```bash
lxc stop ubuntu-test
```

恢复它：

```bash
lxc restore ubuntu-test ubuntu-test-1
```

再次启动容器：

```bash
lxc start ubuntu-test
```

如果再次进入容器并查看，你创建的 "this_file.txt" 现在已不存在。

当你不再需要某个快照时，可以将其删除：

```bash
lxc delete ubuntu-test/ubuntu-test-1
```

!!! warning

    你应该始终在容器运行时删除快照。为什么？因为 _lxc delete_ 命令也可以用于删除整个容器。如果你在上面的命令中在 "ubuntu-test" 后不小心按了回车，并且容器处于停止状态，那么容器将被删除。没有任何警告，它只是按你的要求执行。

    但是，如果容器正在运行，你将收到以下消息：

    ```
    Error: The instance is currently running, stop it first or pass --force
    ```

    因此，请始终在容器运行时删除快照。

在接下来的章节中，你将：

* 设置自动创建快照的过程
* 设置快照过期，使其在特定时间后自动删除
* 设置自动刷新快照到快照服务器
