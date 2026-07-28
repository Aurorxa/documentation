---
title: 崩溃分析
author: Howard Van Der Wal
contributors: Steven Spencer
tested_with: 8,9,10
ai_contributors: Claude (claude-opus-4-6)
tags:
  - crash
  - debugging
  - kdump
  - kernel
  - vmcore
---

**知识水平**：:star: :star: :star:
**阅读时间**：30 分钟

## AI 使用说明

本文档遵循[此处提供的 AI 贡献政策](../contribute/ai-contribution-policy.md)。如果您发现操作说明中有任何错误，请告知我们。

## 简介

当 Linux 内核崩溃时，系统会产生一个称为 `vmcore` 的内存转储。分析此转储通常是确定生产服务器宕机原因的唯一方法。Rocky Linux 为此工作流提供了两个基本工具：`kdump`（在崩溃时捕获 vmcore）和 `crash` 工具（打开转储进行事后分析）。

本指南涵盖了完整流程——从配置 kdump 捕获 vmcore，到使用 crash 命令识别常见崩溃模式（如阻塞任务 panic、互斥锁损坏和 cgroup 死锁）。还涵盖了崩溃调查期间安全收集 sosreport 的方法，以及何时升级内核还是应用变通方案的指导。在深入 vmcore 分析之前了解基本的内核 panic 处理，请参阅[如何处理内核 panic](../troubleshooting/kernel_panic.md)。

## 前置条件

- 一台 Rocky Linux 8、9 或 10 系统。
- Root 或 sudo 访问权限。
- `/var/crash` 中至少有 2 GB 可用磁盘空间用于 vmcore 转储。
- 网络访问权限，以便从 Rocky Linux 仓库安装软件包。

## 设置 kdump 以捕获 vmcore

`kdump` 服务在系统内存中为次要内核保留一部分空间。当主内核崩溃时，`kdump` 引导此次要内核，并使用它将内存内容作为 vmcore 文件写入磁盘^1^。

### 安装 kdump

安装 `kexec-tools` 软件包，该软件包提供 `kdump` 服务：

```bash
dnf install kexec-tools
```

### 配置保留内存

内核必须在引导时为崩溃内核保留内存。检查当前的 `crashkernel` 参数：

```bash
cat /proc/cmdline | grep crashkernel
```

在 Rocky Linux 8 上，`crashkernel` 默认使用 `crashkernel=auto` 机制设置，该机制允许内核自动计算保留内存大小。在 Rocky Linux 9 和 10 上，`crashkernel=auto` 选项被 `kexec-tools` 中的新机制所取代。可以通过以下命令检查默认值：

```bash
kdumpctl get-default-crashkernel
```

在 Rocky Linux 9 上，这将返回 `1G-2G:192M,2G-64G:256M,64G-:512M`。在 Rocky Linux 10 上，这将返回 `2G-64G:256M,64G-:512M`。在 Rocky Linux 8 上，此子命令不可用。

!!! note

    标准交互式安装（使用 Anaconda 安装程序）通过 kdump 安装程序附加组件自动配置 `crashkernel`。云镜像和未包含 kdump 附加组件的 kickstart 安装可能不会在引导命令行中包含 `crashkernel` 参数。

如果 `crashkernel` 在引导命令行中缺失，请使用 `grubby` 添加：

```bash
kdumpctl get-default-crashkernel
grubby --update-kernel=ALL --args="crashkernel=<value from above>"
```

在 Rocky Linux 8 上，`kdumpctl get-default-crashkernel` 不可用，请使用：

```bash
grubby --update-kernel=ALL --args="crashkernel=auto"
```

重启以使更改生效：

```bash
reboot
```

重启后，验证内存是否已保留：

```bash
cat /sys/kernel/kexec_crash_size
```

非零值表示已为崩溃内核保留内存。

### 配置转储位置

默认转储位置是 `/var/crash`。配置文件是 `/etc/kdump.conf`：

```bash
cat /etc/kdump.conf | grep -v "^#" | grep -v "^$"
```

默认配置将转储写入本地文件系统。关键设置包括：

- `auto_reset_crashkernel yes` — 当内存发生变化时自动调整 crashkernel 保留（仅 Rocky Linux 9+）
- `path /var/crash` — 存储 vmcore 的目录
- `core_collector makedumpfile -l --message-level 7 -d 31` — 压缩转储并过滤不必要的页面

!!! note

    `makedumpfile` 中的 `-d 31` 标志过滤零页面、缓存页面、用户数据页面和空闲页面^3^。这显著减小了 vmcore 的大小。对于具有 64 GB RAM 的系统，生成的 vmcore 通常为 1-4 GB，而非完整的 64 GB。

### 启用和验证 kdump

启用并启动 `kdump` 服务：

```bash
systemctl enable --now kdump
```

验证服务正在运行：

```bash
systemctl status kdump
```

在设置了 `crashkernel` 参数并重启后，输出应显示 `Active: active (exited)`，表示崩溃内核已加载。也可以通过以下命令验证：

```bash
kdumpctl status
```

应报告 `Kdump is operational`。

## 安装 crash 工具和 kernel-debuginfo 软件包

`crash` 工具需要 `crash` 软件包和与生成 vmcore 的内核匹配的 `kernel-debuginfo` 软件包^2^。

### 安装 crash

```bash
dnf install crash
```

### 安装 kernel-debuginfo

`kernel-debuginfo` 软件包提供包含完整调试符号的 `vmlinux` 文件。首先，从 vmcore 中识别内核版本（如果是测试，则使用运行中的内核）：

```bash
uname -r
```

使用 `debuginfo-install` 安装匹配的 debuginfo 软件包，该命令自动启用正确的仓库：

```bash
dnf debuginfo-install kernel-core-$(uname -r)
```

这将安装 `kernel-debuginfo` 和 `kernel-debuginfo-common` 两个软件包。`debuginfo-install` 命令自动处理仓库启用，比手动指定 `baseos-debuginfo` 仓库更可靠。

!!! note

    如果 vmcore 是由与当前运行的内核不同的内核版本生成的，请将 `$(uname -r)` 替换为具体的版本字符串。可以通过检查 `makedumpfile` 头信息或对应 sosreport 中的 `uname` 文件从 vmcore 中找到内核版本。

验证 `vmlinux` 文件是否存在：

```bash
ls /usr/lib/debug/lib/modules/$(uname -r)/vmlinux
```

## 使用 crash 工具打开 vmcore

要打开 vmcore，需要提供 `vmlinux` 调试符号和 vmcore 文件的路径：

```bash
crash /usr/lib/debug/lib/modules/<kernel-version>/vmlinux /var/crash/<timestamp>/vmcore
```

例如：

```bash
crash /usr/lib/debug/lib/modules/5.14.0-611.36.1.el9_7.x86_64/vmlinux /var/crash/127.0.0.1-2025-03-09-10:30:00/vmcore
```

当 crash 打开 vmcore 时，会显示一个包含关键信息的头部：

```text
      KERNEL: /usr/lib/debug/lib/modules/5.14.0-611.36.1.el9_7.x86_64/vmlinux
    DUMPFILE: vmcore  [PARTIAL DUMP]
        CPUS: 4
        DATE: Sun Mar  9 10:30:00 JST 2025
      UPTIME: 3 days, 12:45:30
 LOAD AVERAGE: 45.67, 42.31, 38.92
       TASKS: 312
    NODENAME: rocky-server
     RELEASE: 5.14.0-611.36.1.el9_7.x86_64
     VERSION: #1 SMP PREEMPT_DYNAMIC
     MACHINE: x86_64
      MEMORY: 8 GB
       PANIC: "BUG: kernel NULL pointer dereference"
         PID: 0
     COMMAND: "swapper/0"
```

需要检查的关键字段：

- `UPTIME` — 崩溃前系统运行了多长时间
- `LOAD AVERAGE` — 崩溃时的系统负载（高值可能表示资源耗尽）
- `PANIC` — 触发崩溃的 panic 消息
- `PID` 和 `COMMAND` — 崩溃发生时运行的进程和命令

## 基本 crash 分析命令

### `log` — 内核环形缓冲区

`log` 命令显示内核环形缓冲区（相当于 `dmesg`）^4^。这通常是打开 vmcore 后运行的第一个命令：

```text
crash> log
```

要搜索特定消息，通过 `grep` 管道过滤：

```text
crash> log | grep -i "blocked\|panic\|bug\|error"
```

### `bt` — 回溯

显示崩溃发生时活跃任务的回溯：

```text
crash> bt
```

显示特定 PID 的回溯：

```text
crash> bt <pid>
```

显示每个 CPU 上活跃任务的回溯：

```text
crash> bt -a
```

### `ps` — 进程列表

列出所有进程：

```text
crash> ps
```

`-m` 标志显示每个任务在当前状态中花费的时间，这对于识别长时间阻塞的任务至关重要：

```text
crash> ps -m
```

### `foreach` — 遍历任务

`foreach` 命令对多个任务运行命令。要查找所有处于不可中断睡眠（UN 状态）的任务并显示它们被阻塞的时间：

```text
crash> foreach UN ps -m
```

这是诊断阻塞任务 panic 最重要的命令之一。输出显示每个阻塞任务及其在 UN 状态中累计的时间。

### `files` — 打开的文件描述符

显示特定任务打开的文件描述符：

```text
crash> files <pid>
```

### `struct` — 检查内核数据结构

在特定内存地址显示内核结构：

```text
crash> struct task_struct <address>
```

显示特定字段：

```text
crash> struct task_struct <address> | grep pi_blocked_on
```

```text
crash> struct task_struct.pi_blocked_on <address>
```

### `kmem -i` — 内存使用摘要

显示系统内存使用摘要：

```text
crash> kmem -i
```

显示总内存、空闲内存、缓冲区、缓存和交换区使用情况。崩溃时的高内存消耗或交换区使用可能表明内存压力是促成因素。

### `mod -t` — 检查受污染模块

显示内核污染状态以及哪些模块导致了污染：

```text
crash> mod -t
```

被污染的内核（例如，被树外或专有模块污染）的行为可能与上游内核开发者预期的不同。常见的污染标志包括：

- `P` — 加载了专有模块
- `O` — 加载了树外模块
- `E` — 加载了未签名模块

## 识别常见崩溃模式

### 阻塞任务 panic (khungtaskd)

`khungtaskd` 内核线程监控处于不可中断睡眠（D 状态）的任务。如果任务在此状态下的时间超过 `kernel.hung_task_timeout_secs` 阈值（默认：120 秒），`khungtaskd` 会记录警告。如果 `kernel.hung_task_panic` 设置为 1，则会触发内核 panic。

在日志输出中识别该模式：

```text
crash> log | grep "blocked for more than"
```

典型输出：

```text
INFO: task kworker/2:1:1234 blocked for more than 120 seconds.
INFO: task runc:[2:INIT]:5678 blocked for more than 600 seconds.
```

查找所有阻塞的任务：

```text
crash> foreach UN ps -m
```

列出处于不可中断睡眠的每个任务及其持续时间。阻塞数百秒的任务是根因的有力候选。

追踪阻塞链：

确定阻塞任务后，检查其回溯：

```text
crash> bt <pid>
```

在回溯中查找与锁、互斥锁或 I/O 等待相关的函数。常见的阻塞点包括 `mutex_lock`、`rwsem_down_read_slowpath` 和 `io_schedule`。

### 互斥锁损坏 (rt_mutex)

在使用 PREEMPT_RT 的内核上，`spinlock_t` 和 `rwlock_t` 被替换为基于 `rt_mutex` 的实现，将其从自旋锁转换为睡眠锁。这些结构损坏会导致级联任务阻塞。

检查 pi_blocked_on：

如果任务在 rt_mutex 上阻塞，其 `task_struct` 中的 `pi_blocked_on` 字段指向 `rt_mutex_waiter` 结构：

```text
crash> struct task_struct.pi_blocked_on <task_address>
```

如果结果为非 NULL 值，检查 waiter 结构：

```text
crash> struct rt_mutex_waiter <waiter_address>
```

这将显示 `lock` 字段，该字段指向 `rt_mutex` 本身：

```text
crash> struct rt_mutex <mutex_address>
```

`rt_mutex` 的 `owner` 字段显示哪个任务持有该锁。无效的 owner 指针（如 `0x1` 或其他明显无效的地址）表示互斥锁损坏。

损坏的 rt_mutex 链示例：

```text
crash> struct task_struct.pi_blocked_on ffff9a3c0e4b0000
  pi_blocked_on = 0xffff9a3c12340100
crash> struct rt_mutex_waiter 0xffff9a3c12340100
  lock = 0xffff9a3c56780200
crash> struct rt_mutex 0xffff9a3c56780200
  owner = 0x1    <-- 无效指针，表示损坏
```

`owner` 值为 `0x1` 表示锁的所有权跟踪已损坏。此模式在特定 rt_mutex 修复之前的 PREEMPT_RT 内核中观察到。

### cgroup 死锁

容器环境容易受到 cgroup 相关死锁的影响，特别是在容器运行时（如 `runc`）与内核 cgroup 子系统交互时。

识别该模式：

```text
crash> log | grep -i "cgroup\|threadgroup"
```

常见的死锁场景涉及 `cgroup_mutex` 和 `cgroup_threadgroup_rwsem` 锁。一个任务持有 `cgroup_mutex` 同时等待 `cgroup_threadgroup_rwsem`，而另一个任务持有 `cgroup_threadgroup_rwsem` 同时等待 `cgroup_mutex`。

追踪死锁：

1. 查找与容器操作相关的阻塞任务：

    ```text
    crash> foreach UN bt | grep -A 5 "cgroup\|runc"
    ```

2. 通过检查其回溯来识别持有冲突锁的任务。查找函数如 `cgroup_lock`、`cgroup_attach_task`、`copy_process` 和 `cgroup_exit`。

3. 死锁通常涉及：
    - 容器运行时进程（例如 runc）在容器设置期间持有 `cgroup_mutex`。
    - fork 或 exit 操作在 `cgroup_threadgroup_rwsem` 上阻塞。
    - 两个锁形成循环依赖。

缓解措施：

减少触发 cgroup 锁竞争的操作频率——例如 Kubernetes 中的容器 exec 探测——可以防止这些死锁的发生。

### 定时器错误

定时器相关的内核错误表现为定时器代码路径中的 `BUG_ON` 断言。

识别该模式：

```text
crash> log | grep -i "BUG.*timer\|timer.*BUG"
```

```text
crash> bt
```

在回溯中查找函数如 `__run_timers`、`call_timer_fn` 或子系统特定的定时器处理程序（如 SCTP 的 `sctp_generate_timeout_event`）。

定时器错误通常在上游内核补丁中修复。回溯和特定的 `BUG_ON` 消息是搜索已知修复或报告问题时所需的关键信息。

## PREEMPT_RT 内核注意事项

PREEMPT_RT 补丁集将内核的 `spinlock_t` 和 `rwlock_t` 转换为基于 `rt_mutex` 的实现，以提供确定性的调度延迟^5^。在 PREEMPT_RT 下，标准的 `struct mutex` 类型也基于 `rt_mutex` 重新实现，获得优先级继承支持，尽管在两种配置中它们仍然是睡眠锁。此转换显著改变了阻塞行为。

PREEMPT_RT 下的关键区别：

- 自旋锁变为睡眠锁：在标准内核上非阻塞的代码路径在 PREEMPT_RT 下可能阻塞，产生新的死锁可能性。
- 优先级继承：rt_mutex 实现优先级继承，这意味着互斥锁链可能变得更复杂。`task_struct` 中的 `pi_blocked_on` 和 `pi_waiters` 字段被主动使用。
- 更长的阻塞链：由于更多锁是可睡眠的，任务可能被阻塞更长时间，使 `khungtaskd` panic 更可能发生。

RT 内核的额外分析技术：

检查优先级继承链：

```text
crash> struct task_struct.pi_waiters <task_address>
crash> struct task_struct.pi_blocked_on <task_address>
```

检查 rt_mutex 特定字段：

```text
crash> struct rt_mutex <address>
```

在 PREEMPT_RT 内核上，特别关注互斥锁所有权与任务调度优先级之间的关系。如果优先级继承未能正确传播，低优先级任务持有高优先级任务所需的锁可能导致长时间阻塞。

## 在崩溃调查期间安全收集 sosreport

`sosreport` 工具（由 `sos` 软件包提供）收集系统配置和诊断信息^6^。然而，在已经承受压力的系统上运行完整的 sosreport——例如，刚从 panic 中恢复或出现挂起任务的系统——可能触发额外的崩溃。

### 在受压系统上运行完整 sosreport 的风险

完整的 sosreport 运行大量诊断命令，并从 `/proc` 和 `/sys` 读取许多文件。在内核子系统处于不一致状态的系统上，此活动可能：

- 通过访问损坏的数据结构触发额外的内核 panic。
- 导致系统完全无响应。
- 生成不完整且对分析无用的 sosreport。

### 使用有限插件范围

为降低风险，将 sosreport 限制到特定插件：

```bash
sos report -o kernel,process,logs
```

这仅收集内核配置、进程信息和系统日志——通常足以进行初始崩溃调查，而不会给系统带来过大负载。

根据场景有用的其他插件组合：

```bash
sos report -o kernel,process,logs,networking
```

```bash
sos report -o kernel,process,logs,cgroups,container_log
```

### 替代方案：手动收集单个文件

如果即使是有限的 sosreport 风险仍然太大，手动收集必要的文件：

```bash
cp /var/log/messages /tmp/crash_collection/
cp /proc/cmdline /tmp/crash_collection/
cp /etc/kdump.conf /tmp/crash_collection/
uname -a > /tmp/crash_collection/uname.txt
lsmod > /tmp/crash_collection/lsmod.txt
ps auxf > /tmp/crash_collection/ps.txt
cat /proc/meminfo > /tmp/crash_collection/meminfo.txt
```

## 何时升级内核与何时应用变通方案

在确定崩溃的根因后，必须决定是升级内核还是应用变通方案。

### 检查已知修复

搜索内核变更日志中与您识别的崩溃模式相关的修复：

```bash
rpm -q --changelog kernel | grep -i "<search_term>"
```

例如，检查是否包含 rt_mutex 修复：

```bash
rpm -q --changelog kernel | grep -i "rt_mutex\|rtmutex"
```

检查是否有较新内核版本可用：

```bash
dnf check-update kernel
```

### 评估决策

在以下情况升级内核：

- 崩溃模式匹配已知错误，且较新内核版本中有修复。
- 较新版本的 `rpm --changelog` 输出包含相关修复。
- 系统能够容忍用于重启的维护窗口。

在以下情况应用变通方案：

- 尚未有内核修复可用。
- 系统无法承受内核升级的停机时间。
- 可以通过更改系统行为避免崩溃（例如，减少容器 exec 探测频率以避免 cgroup 锁竞争）。

### 验证修复是否包含在内

在确定潜在修复后，验证它是否包含在目标内核版本中：

```bash
rpm -q --changelog kernel-<new_version> | grep -i "<fix_description>"
```

也可以比较当前和可用软件包的内核版本：

```bash
rpm -q kernel
dnf list available kernel
```

## 总结

使用 kdump 和 crash 进行内核崩溃分析为诊断 Rocky Linux 上的生产系统故障提供了系统化的方法。通过配置 kdump 捕获 vmcore、使用 crash 工具检查故障时的内核状态并理解常见的崩溃模式，管理员可以识别根因并应用适当的修复。

关键要点：

- 在崩溃发生前配置 kdump 并验证其正常运行。
- 从 `log`、`bt` 和 `foreach UN ps -m` 开始分析，以了解崩溃上下文。
- 对于阻塞任务 panic，通过互斥锁和锁结构追踪阻塞链。
- 在 PREEMPT_RT 内核上，特别关注 rt_mutex 行为。
- 使用 `-o` 限制插件范围安全收集 sosreport。
- 在升级前使用内核变更日志验证修复是否包含在内。

## 参考资料

1. "Documentation for kdump — The kexec-based Crash Dumping Solution"，The Linux Kernel documentation project [https://docs.kernel.org/admin-guide/kdump/kdump.html](https://docs.kernel.org/admin-guide/kdump/kdump.html)
2. "crash utility"，the crash-utility project [https://github.com/crash-utility/crash](https://github.com/crash-utility/crash)
3. "makedumpfile"，the makedumpfile project [https://github.com/makedumpfile/makedumpfile](https://github.com/makedumpfile/makedumpfile)
4. "crash(8) man page"，the crash-utility project [https://man7.org/linux/man-pages/man8/crash.8.html](https://man7.org/linux/man-pages/man8/crash.8.html)
5. "Real-Time Linux"，The Linux Foundation [https://wiki.linuxfoundation.org/realtime/start](https://wiki.linuxfoundation.org/realtime/start)
6. "sos — A unified tool for collecting system logs and other debug information"，the sos project [https://github.com/sosreport/sos](https://github.com/sosreport/sos)
