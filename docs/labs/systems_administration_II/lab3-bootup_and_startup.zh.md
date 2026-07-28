---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - lab exercise
  - boot
  - startup
  - systems administration
---

# 实验 3: 启动与开机流程

!!! info

    输入命令 `lab3-bootup_and_startup`，启动一个名为 `lab3-bootup_and_startup` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，你将了解 Rocky Linux 系统的启动和开机过程。Linux 启动过程是系统启动引导、初始化服务并为用户交互做好准备的过程。我们将探索从按下电源按钮或不带电源按钮的虚拟机到登录提示的整个过程，并进行亲身实践。通常，这被称为引导过程。

    !!! knowledge "知识要点"

        本实验将带我们了解 Rocky Linux 系统启动过程的 6 个标准阶段和关键文件/实用程序，即：

        1. BIOS / UEFI
        2. 引导加载程序阶段（GRUB2）
        3. 内核初始化
        4. `initramfs`
        5. `systemd`
        6. 登录

        在此过程中，我们将检查每个步骤中涉及的重要文件。你的任务是确保你理解了启动过程的每个标准阶段，并通过实际操作练习来掌握它们。

## 目标

1. 识别 Rocky Linux 启动过程的关键阶段。
2. 检查和修改与启动相关的配置文件。
3. 使用 `systemd` 目标（target）和服务。

## 先决条件

* 1 台安装了 Rocky Linux 的机器
* 具有 root 访问权限或 `sudo` 凭据

## 启动流程

### GRUB 引导加载程序

1. 以 root 用户身份检查 `/etc/default/grub` 文件的内容。

    !!! question "问题"

        解释 `GRUB_TIMEOUT` 参数的含义。

2. 将超时设置为 10 秒，然后重新生成 GRUB 配置文件：

    ```bash
    grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

3. 使用 `grubby` 工具查看默认内核和所有已安装的内核。

    !!! question "问题"

        你需要使用哪些标志来获取默认内核的引导信息？你在工作中会用什么标志将默认内核改为其他版本？

4. 查看启动时传递给内核的命令行参数。

    !!! question "问题"

        这些参数位于哪个文件或工具中？如何修改它们？

### 内核和 initramfs

1. 列出 `/boot` 目录的内容。你看到了什么？你能分辨出内核文件、`initramfs` 文件和 System.map 文件吗？
2. 使用 `lsinitrd` 工具检查 `initramfs` 文件。示例：

    ```bash
    lsinitrd /boot/initramfs-$(uname -r).img
    ```

    !!! question "问题"

        1. 解释 initramfs 文件的作用，并说明它与 initrd 的区别。
        2. 你可能通过哪些方法来确定运行系统中的内核版本？包括使用 `/proc` 文件系统的方法。

### systemd 和启动目标（target）

1. 检查你的 Rocky Linux 系统正在运行的默认 `systemd` 目标。

    !!! question "问题"

        将系统的默认启动目标设置为 `multi-user.target`。你需要运行什么命令？使用 `multi-user.target` 与使用 `graphical.target` 的区别是什么？

2. 列出所有活动的 systemd 目标。
3. 检查并显示默认目标所依赖的服务和单元。检查该目标是拉入（pulls in）还是需要（requires）各种服务和目标。

    !!! question "问题"

        你用什么命令来列出服务单元文件中的依赖项？

4. 将默认启动目标更改为 `graphical.target`，然后改回 `multi-user.target`。

### 启动过程的故障排除

1. 使用 systemd 分析启动过程，看看系统启动花了多长时间，以及哪些服务花费了多长时间：

    ```bash
    systemd-analyze
    ```

2. 列出启动时间最长的服务：

    ```bash
    systemd-analyze blame
    ```

3. 使用 journalctl 检查启动日志。

## 总结

!!! knowledge "知识要点"

    你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
