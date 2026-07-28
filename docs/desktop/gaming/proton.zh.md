---
title: 使用 Proton 在 Linux 上游戏
author: Joseph Brinkman
contributors: Steven Spencer, Ganna Zhyrnova
---

## 介绍

Proton 是 Valve 的一个项目，旨在将游戏带入 Linux 桌面的 Steam 客户端中。Proton 是 [Wine](https://www.winehq.org/) 的一个分支（fork），Wine 是一个兼容层，用于在 Linux（及其他 POSIX 兼容操作系统）上运行 Windows 应用程序。

自 2018 年 8 月 Proton 诞生以来，已有 802 条评论发布在 [Proton Compatible Steam Group](https://store.steampowered.com/curator/33483305-Proton-Compatible/about/) 上！这是 Valve 和 Proton 社区取得的巨大进步，因为最初发布时只有 27 款游戏经过测试和认证。

与 Wine 不同，Proton 通常不需要配置，旨在为完全初学者设计。只需安装 Steam 并启用 Proton！

## 假设

* 具有桌面环境的 Rocky Linux Workstation
* Flatpak
* Steam 帐户

## 安装 Steam

使用 Flatpak 安装 Steam：

```bash
flatpak install steam 
```

输入选项 `20` 选择 `app/com.valvesoftware.Steam/x86_64/stable`，然后按 ++enter++ 安装 Steam。

![Installing Steam option 20](images/Timeline_1_01_00_22_00.jpg)

安装 Steam 后，它将自动开始更新。

![Steam updates](images/Timeline_1_01_04_16_00.jpg)

更新后，您必须登录您的 Steam 帐户。如果您没有帐户，应该注册一个。

![Steam](images/Timeline_1_01_06_09_04.jpg)

## 启用 Proton 兼容性

登录 Steam 后，点击左上角的 ++"Steam"++，然后选择 ++"Settings"++。

![Steam settings](images/Timeline_1_01_10_18_38.jpg)

在 Steam 设置中，从左侧菜单中选择 ++"Compatibility"++。

![Compatibility settings](images/Timeline_1_01_10_58_27.jpg)

请注意上图，其中"Enable Steam Play for supported titles"显示为已启用，而"Enable Steam Play for all other titles"未启用。这意味着已测试并验证可在 Proton 上运行的游戏可以玩了，但未经验证的游戏将无法运行。一些未经验证的游戏在 Proton 上运行完美，但需要手柄映射或一些相对较小的调整。因此，作者建议为不受支持的游戏启用 Steam Play，并自行测试！

切换"Enable Steam Play for all other titles"。在提示时重新启动 Steam。

![Steam play for all other titles toggled](images/Timeline_1_01_11_07_44.jpg)

!!! warning "兼容性并非总是普遍适用"

    许多人报告 ProtonDB 兼容游戏出现问题，而原生 Linux 兼容游戏运行正常。这通常是由于 SELinux 策略保护文件系统造成的。

    首先，检查系统的 SELinux 状态：

    ```
    sestatus
    ```
    
    这将返回三种结果之一：

    * SELinux status:                 disabled（如果 SELinux 保护已完全关闭）
    * SELinux status:                 permissive（如果 SELinux 处于宽容模式）
    * SELinux status:                 enforcing（如果 SELinux 正在全面保护您的系统）

    如果 SELinux 被禁用，则它不是导致游戏问题的原因。如果它处于 enforcing 模式，那么它可能是罪魁祸首。请在游戏前尝试将 SELinux 临时设置为 permissive 模式：

    ```
    sudo setenforce 0
    ```

    游戏结束后，记得将 SELinux 策略恢复为 enforcing：

    ```
    sudo setenforce 1
    ```

    对于更永久的解决方案，在保持 SELinux 策略的情况下，您必须研究是哪些规则阻止了您的游戏，这需要对 SELinux 及其底层工具有更深入的了解。请查看[我们的 SELinux 安全指南](../../guides/security/learning_selinux.md)以获得更全面的了解。

## 总结

重启 Steam 后，下载您最喜欢的 Windows 游戏并尝试一下！无需进一步配置。祝您游戏愉快！
