---
title: 安装 Terminator 终端模拟器
author: Andrew Scott
contributors: Steven Spencer, Ganna Zhyrnova
tested with: 9.4
---

!!! warning "暂时搁置"

    这些说明不适用于 Rocky Linux 10。原因是 EPEL 尚未为 10 构建 Terminator。他们*可能*仍会构建它，但 Terminator 项目自 2020 年转移到 GitHub 后，开发甚少，近乎停滞。文档团队将在一段时间内继续测试该软件包，看看是否变得可安装，但可能会在未来删除此页面。

## 简介

Terminator 是一款基于 GNOME Terminal 的终端模拟器，支持多终端面板、终端分组以及保存首选布局等高级功能。

## 前提条件

* 拥有带 GUI 的 Rocky Linux 工作站或服务器
* 具有 `sudo` 权限的管理员身份

## 安装 Terminator

Terminator 位于 Extra Packages for Enterprise Linux (EPEL) 仓库中，在新安装中不可用。因此，首先需要将 EPEL 添加到 Rocky Linux。

* 步骤 1（可选）：启用 CodeReady Builder (CRB) 仓库

```bash
sudo dnf config-manager --set-enabled crb
```

虽然对于 Terminator 并不严格必要，但 CRB 为 EPEL 中某些软件包提供了依赖项，这在计划将来依赖该仓库时会很有用。

* 步骤 2：添加 EPEL 仓库

```bash
sudo dnf install epel-release -y
```

* 步骤 3（可选但强烈推荐）：更新系统

```bash
sudo dnf update -y --refresh
```

* 步骤 4：安装 Terminator

```bash
sudo dnf install terminator -y
```

## 配置

默认情况下，Terminator 与默认的 GNOME Terminal 看起来差别不大。它看起来甚至比默认终端*更加*简洁。

![Terminator 默认布局](images/terminator-01.png)

要开始自定义新终端，在背景任意处右键点击打开上下文菜单。

![Terminator 上下文菜单](images/terminator-02.png)

从该菜单中可以拆分窗口、打开新标签页以及切换布局。"Preferences" 子菜单还允许自定义主题。值得花些时间熟悉可用选项，因为有许多设置超出了本指南的范围。

还有多个快捷键绑定，适合那些不喜欢在键盘和鼠标之间来回切换的用户。例如，++shift+ctrl+"O"++ 可以水平分割窗口为多个终端。还支持多次分割窗口和拖放重新排列。

![带有 3 个分割终端的 Terminator 窗口](images/terminator-03.png)

最后，设置键盘快捷键来打开新终端也很有帮助。为此，可以从打开"设置"(Settings) 菜单开始。可以通过几种不同方式访问菜单，本指南将在桌面右键点击并左键点击 "Settings"。

![桌面上下文菜单，高亮显示 "Settings"](images/terminator-04.png)

从这里，使用左侧菜单导航到 "Keyboard" 部分，然后点击底部的 "Customize Shortcuts"。

![GNOME 设置键盘菜单](images/terminator-05.png)

如果这是首次设置自定义快捷键，将看到一个标题为 "Add Shortcut" 的按钮。否则，会看到快捷键列表，底部有一个加号。点击适用于您情况的选项，打开 "Add Custom Shortcut" 对话框。在 *Name* 字段中，输入一个易于记忆的快捷键别名。在 *Command* 字段中，输入程序名称：`terminator`。然后点击 "Set Shortcut" 设置新的按键组合。

![添加自定义快捷键对话框](images/terminator-06.png)

虽然 ++ctrl+alt+"T"++ 是传统选择，但可以自由选择任意组合。以后随时可以更新快捷键名称和按键组合。要保存快捷键，点击 "Add Custom Shortcut" 对话框右上角的 "Add"。

![已完成 Terminator 的自定义快捷键对话框](images/terminator-07.png)

## 结语

Terminator 是一款功能强大的终端模拟器，适合普通用户和高级用户。这些示例仅展示了 Terminator 能力的一小部分。虽然本指南提供了 Rocky Linux 上的安装步骤概览，但您可能需要查阅 [文档](https://gnome-terminator.readthedocs.io/en/latest/) 以获取 Terminator 功能的完整说明。
