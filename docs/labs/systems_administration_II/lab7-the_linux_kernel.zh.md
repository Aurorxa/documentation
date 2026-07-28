---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - lab exercise
  - kernel
  - systems administration
---

# 实验 7: Linux 内核

!!! info

    输入命令 `lab7-the_linux_kernel`，启动一个名为 `lab7-the_linux_kernel` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，你将深入了解 Linux 内核，它是你所知道的这个著名操作系统的核心。你将练习使用多种工具和命令来理解和管理内核。具体来说，你将：

    * 检查内核版本和已安装的内核
    * 更新内核
    * 管理内核模块
    * 调整内核参数
    * 使用 `/proc` 和 `/sys` 文件系统

    !!! knowledge "知识要点"

        操作系统的内核直接与硬件交互，并管理系统组件之间的交互。它在内存中为程序分配独立的区域，防止一个程序侵犯另一个程序的空间。此外，Linux 内核负责根据进程的需求和系统负载确定每个进程运行的时长。不同的进程可能以不同的优先级执行——内核负责为这些进程分配不同的调度优先级。

        在本实验中，你将花大量时间使用这些工具，并执行涉及 Linux 内核的操作。

## 目标

1. 查看本地系统中可用的 Linux 内核。
2. 安装不同版本的 Linux 内核。
3. 通过可加载模块扩展 Linux 内核。
4. 通过内核参数调整内核行为。

## 先决条件

* 1 台安装了 Rocky Linux 9.x 的机器
* 具有 root 访问权限或 sudo 凭据

## 检查内核

1. 使用 `uname` 命令查看有关当前运行内核的详细信息。
2. 查看 `/proc/version` 的内容。
3. 检查 `/etc/os-release` 的内容。
4. 列出 `/boot` 目录中与内核相关的所有文件（vmlinuz、initramfs、System.map）。
5. 检查可用的内核：

    ```bash
    dnf list kernel
    ```

    ```bash
    grubby --info=ALL
    ```

    !!! question "问题"

        在你的 Rocky Linux 9.x 系统上，除了默认内核外，还安装了哪些额外内核？

## 使用内核模块

1. 列出所有当前加载的内核模块：

    ```bash
    lsmod
    ```

2. 使用 `modinfo` 获取特定模块的详细信息（例如，ext4）：

    ```bash
    modinfo ext4
    ```

3. 将一个已加载的模块从内核中移除，然后稍后重新加载。

    !!! question "问题"

        有些模块不能被移除，因为它们正在使用。如何获取某个特定模块的使用计数？

4. 加载一个目前未加载的模块。
5. 在实验室结束时，验证所有你应该卸载的模块已经被卸载。

## 使用内核参数调优内核

Linux 内核可以通过运行时可调参数进行配置。你可以通过多种方式修改这些参数。

### 通过命令行使用 sysctl

* 临时修改内核参数（`net.ipv4.ip_forward`）的值。
* 查看该值是否修改成功。
* 将修改持久化到 `/etc/sysctl.d/` 目录中。
* 使内核命令行工具更方便使用，例如启用 tab 自动补全。如何操作？

### 通过 proc 和 sysfs 文件系统

* 查看当前的最大共享内存大小：

    ```bash
    cat /proc/sys/kernel/shmmax
    ```

* 通过同样位于 `/proc/sys/` 下的 `/sys` 接口，查看与网络相关的内核参数的当前设置：

    ```bash
    cat /proc/sys/net/ipv4/ip_forward
    ```

* 同样，如果直接通过 `/proc` 接口修改参数需要谨慎。

!!! question "问题"

    除了通过命令行直接设置外，修改内核参数还有哪些方法？

## 总结

!!! knowledge "知识要点"

    你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
