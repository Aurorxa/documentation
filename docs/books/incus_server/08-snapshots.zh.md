---
title: 8 容器快照
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 9.4
tags:
  - incus 
  - enterprise
  - incus snapshots
---

在本章的全部内容中，你需要以你的非特权用户身份运行命令（如果你从一开始就一直在跟随本手册，则为 "incusadmin"）。

容器快照和快照服务器（稍后详述）是运行生产级 Incus 服务器最关键的方面。快照可确保快速恢复。在更新运行在特定容器上的主要软件时，将其用作故障安全机制是一个好主意。如果在更新过程中发生某些事情破坏了该应用程序，你只需恢复快照，就可以在几秒钟的停机时间内恢复运行。

作者使用了 Incus 容器来运行 PowerDNS 面向公众的服务器，由于在每次更新之前都进行了快照，更新这些应用程序变得不那么有问题了。

你甚至可以在容器运行时对其进行快照。

## 快照过程

首先，使用此命令获取 ubuntu-test 容器的快照：

```bash
incus snapshot create ubuntu-test ubuntu-test-1
```

这里，你将快照称为 "ubuntu-test-1"，但你可以随意命名。要确保你拥有快照，请对容器执行 `incus info`：

```bash
incus info ubuntu-test
```

你已经查看过信息屏幕。如果你滚动到底部，现在会看到：

```bash
Snapshots:
  ubuntu-test-1 (taken at 2021/04/29 15:57 UTC) (stateless)
```

成功！我们的快照已就位。

进入 ubuntu-test 容器：

```bash
incus shell ubuntu-test
```

使用 _touch_ 命令创建一个空文件：

```bash
touch this_file.txt
```

退出容器。

在将容器恢复到创建文件之前的状态之前，恢复容器最安全的方法（特别是在有许多更改的情况下）是先停止它：

```bash
incus stop ubuntu-test
```

恢复它：

```bash
incus snapshot restore ubuntu-test ubuntu-test-1
```

再次启动容器：

```bash
incus start ubuntu-test
```

如果你再次回到容器并查看，你创建的 "this_file.txt" 已经消失了。

当你不再需要快照时，可以将其删除：

```bash
incus snapshot delete ubuntu-test ubuntu-test-1
```

!!! warning

    你应该始终在容器运行时删除快照。为什么？嗯，_incus delete_ 命令也可用于删除整个容器。如果我们在上面的命令中在 "ubuntu-test" 之后意外按下了回车键，并且如果容器已停止，容器将被删除。我只是想让你知道，不会给出任何警告。它只是执行你要求的内容。

    但是，如果容器正在运行，你将收到此消息：

    ```
    Error: The instance is currently running, stop it first or pass --force
    ```

    因此，始终在容器运行时删除快照。

在接下来的章节中，你将：

* 设置自动创建快照的过程
* 设置快照的过期时间，使其在特定时间长度后消失
* 设置快照自动刷新到快照服务器
