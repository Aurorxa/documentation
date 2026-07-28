---
title: 安装 Kitty 终端模拟器
author: Alex Zolotarov
contributors: Steven Spencer
tested with: 9
---

## 简介

**Kitty 是一个速度极快、功能超强的终端模拟器**，几乎任何您能想象到的内容都可以在 Kitty 中定制。
您可以使用标签管理、平铺布局、图像预览以及其他众多选项，所有这些都在这一个终端模拟器内完成。
甚至可以取代 `tmux` 甚至是窗口管理器（如果您主要在终端中工作的话）。

## 前提条件

- 拥有带 GUI 的 Rocky Linux 工作站或服务器
- 具有 `sudo` 权限的管理员身份

## 安装 Kitty

**首先，安装 EPEL 仓库 (Extra Packages for Enterprise Linux)：**

```bash
dnf install -y epel-release
```

接下来，安装 **Kitty**：

```bash
dnf install -y kitty
```

安装完成后，即可启动 Kitty。

## 快速概览

### 配置文件

启动 **Kitty** 后，可以通过 ++ctrl+shift+f2++ 打开 Kitty 配置文件。
也可以在 `$HOME/.config/kitty` 找到配置文件。

本文档不会深入探讨配置文件本身。只需要知道，您可以在配置中更改任何默认快捷键或任何与外观相关的设置。

### 标签页

可以通过 ++ctrl+shift+t++ 创建新标签页。

可以通过 ++ctrl+shift+w++ *或* ++ctrl+shift+q++ 关闭标签页。

可以通过 ++ctrl+shift+left++ *或* ++ctrl+shift+right++ 选择标签页。

![kittytabs](./images/kitty_tabs.png)

### 平铺布局

按 ++ctrl+shift+enter++ 打开新窗格或窗口。
可以多次按下以创建平铺布局。

可以通过 ++ctrl+shift+l++ 切换布局。

可以通过 ++ctrl+shift+bracket-left++ 或 ++ctrl+shift+bracket-right++ 选择窗口或窗格。
也可以直接用鼠标点击窗格或窗口。

![kittytiling](./images/kitty_tilling.png)

## 结语

Kitty 无需额外配置即可提供许多功能。
如果您的工作站上已经配置了窗口管理器、`zsh` 或 `tmux`，可能不需要 Kitty。但请考虑这样一个事实：您可以仅在一个终端模拟器中结合 `zsh` 快捷键、`tmux` 平铺布局和许多窗口管理器功能。
不过，如果您还没有尝试过这些高级工具，**作者**强烈建议您从 Kitty 开始。
