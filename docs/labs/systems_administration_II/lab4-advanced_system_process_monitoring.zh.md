---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - lab exercise
  - process monitoring
  - systems administration
---

# 实验 4: 高级系统进程监控

!!! info

    输入命令 `lab3-advanced_system_process_monitoring`，启动一个名为 `lab4-advanced_system_process_monitoring` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，我们将更深入地探讨系统中进程的运行机制。使用标准、具有代表性的 `top`（table of processes）进程监控软件，你将学习如何诊断和修复导致高负载的进程、将进程移至后台运行、监控后台进程，以及处理失控的失控进程或恶意程序。

    !!! knowledge "知识要点"

        本实验将介绍如何识别、操作和强制终止失控进程的技术。主要内容包括：

        * 管理失控进程
        * 通过 /proc 文件系统识别进程
        * 使用 `nice`、`renice`、`pidof`、`pgrep`、`pkill`、`kill` 等命令
        * 进程调度
        * `cgroups` 与进程管理
        * 排查和修复高负载场景

        在实操实验中，你将创建场景并使用工具来查找和修复问题。

## 目标

完成本实验后，你应该能够：

* 识别和管理失控进程
* 使用不同的信号终止进程
* 管理进程优先级
* 对 Linux 控制组（control groups，cgroups）有基本的理解
* 使用各种进程监控工具（top、ps、htop）识别高负载情况

## 先决条件

* 1 台安装了 Rocky Linux 的机器
* 具有 root 访问权限或 sudo 凭据
* 可选：安装 `htop`

## 失控进程

失控进程是一种持续消耗资源（CPU、内存或 I/O），直到系统变得反应迟缓或完全无响应的进程。进程失控的原因有很多，包括：

* 软件缺陷导致死循环
* 内存泄漏
* 硬件故障
* 恶意软件

!!! question "问题"

    你将如何使用 `top` 或 `ps` 识别 CPU 或内存使用率最高的进程？解释使用的具体选项。

### 实验场景 A：创建并管理失控进程

1. 生成一个会消耗大量 CPU 资源的进程：

    ```bash
    dd if=/dev/zero of=/dev/null &
    ```

    这创建了一个无限读取 `/dev/zero` 并写入 `/dev/null` 的进程。注意 shell 返回的 PID。

2. 使用 `top` 确认该进程占用了接近 100% 的 CPU。
3. 终止该进程，使用以下两种方式：
    * 从 `top` 内部（按 `k`，输入 PID，然后输入信号 15）
    * 从命令行使用 `kill -15 <PID>`
4. 再次运行上面的 `dd` 命令。
5. 使用 `kill -9 <PID>` 终止该进程。
6. 现在生成两个会消耗 CPU 的进程，并将它们放入后台。确保它们都在运行。

    !!! question "问题"

        1. 如何将前台进程挂起（暂停）并放入后台？
        2. 如何将后台任务调回前台？
        3. `kill -9`（SIGKILL）和 `kill -15`（SIGTERM）的区别是什么？

## 使用 /proc 文件系统进行进程管理

`/proc` 文件系统包含有关系统进程的信息。每一正在运行的进程都由 `/proc` 下的一个目录表示，目录名为进程 ID（PID）。

!!! question "问题"

    如何使用 `/proc` 文件系统识别消耗最多内存的进程？

## 进程调度与 nice 值

Linux 调度器决定哪些进程获得 CPU 时间以及获得多长时间。进程的优先级由 `nice` 值决定，范围从 -20（最高优先级）到 19（最低优先级）。

1. 查看正在运行的进程的 nice 值。
2. 创建一个 nice 值为 10 的后台进程。
3. 将其 nice 值更改为 15。

    !!! question "问题"

        1. 如何查看系统中所有进程的 nice 值？需要显示包含 nice 值的特定列。
        2. `renice` 和 `nice` 命令有什么区别？何时分别使用它们？

## 信号与进程控制

信号是发送给进程的消息，通知其某个事件已经发生。信号可用于终止、挂起或指示进程执行特定操作。

```bash
kill -l
```

这将列出所有可用的信号。

1. 使用信号终止进程时，使用正确的语法，传递信号名称及其数值。**练习**：尝试使用信号 `-9`（又称 `SIGTERM`）终止进程。

    !!! question "问题"

        你如何列出 Linux 系统上可用的信号，并解释常见信号的作用？至少列出 6 个常见信号。

## 控制组（cgroups）

控制组（cgroups）是一项 Linux 内核特性，用于限制、记录和隔离进程组的资源使用情况（CPU、内存、磁盘 I/O 等）。

在 Rocky Linux 9.x 上，cgroups v2 是默认的。

1. 检查你的系统是使用 cgroups v2 还是 v1：

    ```bash
    mount | grep cgroup
    ```

2. 探索 cgroups 的文件系统层次结构，导航到 `/sys/fs/cgroup/` 并查看：CPUs、内存、I/O。

3. 使用 `systemd-cgtop` 监控控制组的资源使用情况。

## 排查和修复高负载场景

你将在服务器上解决多个性能问题。你收到的告警如下：

### 场景：CPU 使用率过高

```bash
# 模拟 CPU 过载
stress-ng --cpu 2 --timeout 60s
```

按以下步骤排查此高 CPU 使用率问题：

* 找到高负载的进程
* 通过管理优先级将其降低
* 将该进程限制在特定的 CPU 上

!!! question "问题"

    1. 你将使用什么命令将运行中的进程限制在特定的 CPU 核心（例如，核心 0 和 1）？
    2. 解释在共享或虚拟化环境中，当多个进程同时争抢 CPU 时，可能出现的问题以及管理员如何缓解。

## 总结

!!! knowledge "知识要点"

    你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
