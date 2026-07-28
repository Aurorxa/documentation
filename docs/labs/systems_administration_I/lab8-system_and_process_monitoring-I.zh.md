---
author: Wale Soyinka
contributors: Steven Spencer
tags:
  - lab exercise
  - process monitoring
  - systems administration
---

# 实验 8: 系统与进程监控 I

!!! info

    输入命令 `lab8-system_and_process_monitoring-one`，启动一个名为 `lab8-system_and_process_monitoring-one` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，你将深入研究性能监控，以及用于显示和管理服务器上进程资源的常见命令行工具。

    !!! knowledge "知识要点"

        本实验将让你掌握用于检查和管理系统上各级别进程的命令，主要是通过 `/proc` 文件系统目录。不过，所有其他可用的工具也都已为你准备就绪！

        你将逐步练习使用各种监控工具，例如：

        * 使用 `top` 和 `ps` 进行进程监控和管理
        * 使用 `vmstat`、`iostat`、`sar`、`free` 和 `dstat` 等监控系统性能和资源

## 目标

* 使用各种命令行工具（如 `top`、`ps`、`kill`、`iostat`、`vmstat` 等）监控和管理进程
* 使用文件系统 `/proc` 来识别和管理进程

## 先决条件

在开始本实验之前，你需要：

* 1 台安装了 Rocky Linux 的机器
* 具有 root 访问权限或 sudo 凭据
* 安装实验需要的所有额外软件包：`sysstat`、`procps`、`util-linux`

## 进程监控

### 使用 `/proc` 文件系统

`/proc` 文件系统是一个伪文件系统，用作 Linux 内核内部数据结构的接口。它包含有助于监控进程和系统信息的文件。

让我们探索 `/proc` 文件系统的一些关键组件。

1. 查看与 CPU 相关的信息——例如，通过显示 `/proc/cpuinfo` 文件的内容。
2. 查看内存信息——例如，通过显示 `/proc/meminfo` 文件的内容。
3. 查看块设备信息——例如，通过查看 `/proc/diskstats`。
4. 选择当前正在运行的进程的 PID，然后使用 `ls` 浏览其 `/proc/<PID>/` 目录，了解有哪些可用信息。
5. 常见的进程特定文件——例如，要显示 `systemd`（PID 1）进程的命令行参数，可以运行：

    ```bash
    cat /proc/1/cmdline
    ```

    !!! question "问题"

        1. 你会使用什么来读取与进程的实时环境变量对应的文件？
        2. 如何从 `/proc` 文件系统中获取以下信息：
            * 内核版本
            * 系统的正常运行时间
            * 系统上支持的文件系统
            * 系统的平均负载

### `top`

`top` 程序提供了一个动态的、实时的系统进程视图。

1. 以最基本的形式运行 `top`。在 top 运行时熟悉以下交互命令：
    * 按内存使用量排序（`M`）
    * 按 CPU 使用率排序（`P`）
    * 按用户过滤（`u`）
    * 仅显示特定用户的进程（例如，`top -u root`）
    * 杀死一个进程（`k`）
    * 自定义显示列

    !!! question "问题"

        1. 你会使用哪个 `top` 命令来仅显示用户 `nobody` 的进程？
        2. 如何使用 `top` 更改内存和 CPU 的显示单位（例如，从 KB 更改为 MB、GB）？
        3. 将当前 `top` 显示输出导出到文本文件中。

2. 找到系统中 CPU 使用率最高的前 5 个进程。
3. 完成以下任务，然后解释其意义和影响：
    * 将正在运行的 `top` 实例中的刷新延迟更改为 10 秒。
    * 对 `top` 中的 `PR`（优先级）和 `NI`（nice 值）列进行排序。
    * 按 `PR` 排序时，低数值和高数值分别代表什么？
    * 如何解释 `top` 输出中的 `load average` 字段？
    * 使用 `top` 向进程发送信号（如 15 或 9）并杀死该进程。

### `ps`

`ps` 命令提供当前进程的快照。

1. 以几种不同的格式运行 `ps` 命令。
2. 以分层视图显示所有进程。
3. 显示特定用户拥有的所有进程。
4. 显示所有进程的线程。
5. 使用 `ps` 和 `grep` 的组合查找特定进程，如 `sshd`。然后使用 `kill` 或 `pkill` 结束该进程。
6. 执行以下任务并回答问题：

    !!! question "问题"

        1. `ps aux` 输出中的进程状态代码（如 `S`、`R`、`D`、`Z`、`T`）分别表示什么含义？在系统上查找每种状态代码的示例。
        2. 你将使用什么命令来识别系统中僵尸进程的 PID 和数量？

### `pgrep`

1. 使用 `pgrep` 查找 `sshd` 和相关 SSH 进程的 PID。
2. 用 `pgrep` 查找属于特定用户（例如 root）的所有进程，并显示它们的 PID。
3. 运行一个包含 `pgrep` 和 `pkill` 的命令，对于系统运行并不重要的进程，先找到其 PID，然后将其杀死。确保你有一个可安全杀死的不必要进程。

!!! question "问题"

    如果你知道一个进程的 PID，如何找出其所有子进程？提示：`-P`。

## 系统监控

### `free`

1. 使用 `free` 显示有关已用和可用系统内存的信息。
2. 以人类可读的格式（例如 MB 和 GB）查看内存输出。

    !!! question "问题"

        如何解释 `free` 输出中的 `Shared`、`buff/cache` 和 `available`？

### `vmstat`

1. 单独运行 `vmstat`，查看系统资源的摘要。
2. 运行 `vmstat` 5 次，间隔 5 秒，以跟踪一段时间内的性能。
3. 运行 `vmstat`，带上显示磁盘统计信息的选项。

    !!! question "问题"

        在 `vmstat` 输出中，`procs` 下的 `r` 列表示什么？`b` 列又表示什么？

### `iostat`

1. 显示基本的 CPU 和 I/O 统计信息。
2. 显示特定磁盘的性能统计信息。
3. 以 2 秒的间隔显示 I/O 统计信息，共显示 5 次。

    !!! question "问题"

        在 `iostat` 输出中，`iowait` 列表示什么？持续的高 `iowait` 值表明什么？

### `sar`

`sar` 命令收集、报告和保存系统活动信息。安装 `sysstat` 包后，验证：

1. `sysstat` 服务已启用并正在运行。
2. 查看 `sysstat` 用于保存其信息的配置文件路径。
3. 查看 CPU 使用率统计信息。示例 `sar -u`。
4. 查看内存使用率统计信息。

    !!! question "问题"
        1. 如何查看特定时间段内（例如 09:00 到下午 1:00）的 `sysstat` sar 数据？
        2. 如何以不同于默认值（例如 `5` 次，间隔 `2` 秒）的频率收集 sar 指标？
        3. 如何更改 `sysstat` 保存其日志文件的位置？例如，你希望在运行 Rocky Linux 的机器上重定向到 `/var/log/custom_sa/` 目录。

### `dstat`

`dstat` 是一种多功能的系统资源统计替代工具。

1. 安装并运行 `dstat`，使用 CPU、磁盘和网络统计信息选项。
2. 将 `dstat` 的输出写入文件。

    !!! question "问题"

        `dstat` 相对于 `vmstat`、`iostat` 等工具有什么优势？

### 综合任务

1. 使用 `top` 或 `ps` 找到 CPU 或内存使用率最高的进程。注意其 PID。
2. 暂停该进程，并使用 `top` 或 `ps` 验证其状态。
3. 终止该进程，并确认其已不再运行。
4. 检查 `/proc/meminfo` 和 `/proc/cpuinfo`，查看系统健康状况和运行情况。

## 总结

!!! knowledge "知识要点"

    你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
