---
title: Micro 编辑器安装
author: Joseph Brinkman, Steven Spencer
contributors: Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - 编辑器
  - micro
---

# Micro 终端文本编辑器

本指南提供了如何在 Linux 或 macOS 上安装 Micro 编辑器的清晰说明。

## 介绍

Micro 是一个基于终端的开源文本编辑器，旨在通过其丰富的功能集替代 `nano`。Micro 是一个单一的、独立的静态二进制文件，易于安装和使用。开箱即用，只需运行 `micro` 即可开始编辑。

Micro 提供许多功能，包括：

- 对常用按键绑定(Ctrl+S、Ctrl+Z 等)的直观支持。
- 全面的鼠标支持。这意味着您可以使用鼠标进行文本选择或窗口调整大小。
- 真正的颜色支持（通过终端在 micro 中设置颜色主题）。
- 支持多个游标编辑。
- 支持水平和垂直分割。
- 为超过 130 种语言内置语法高亮。
- 以及一个插件系统等。

## 前提条件

* Rocky Linux 工作站或服务器。启动为 root 用户直接执行命令，或能够通过 `sudo` 提权执行命令。
* 具备熟练的命令行操作能力，并能使用 `vi`、`nano` 等编辑器。

## 如何安装 Micro

### `curl` 脚本

要快速安装 `micro`，您可以通过运行以下命令，从脚本安装最新的预编译二进制文件：

```bash
curl https://getmic.ro | bash
```

脚本将 `micro` 安装到当前目录。从那里，将其移动到您的路径中，例如 `/usr/local/bin`。

要安装剪切板支持，您需要一些额外的软件包：

```bash
dnf install xclip xsel
```

请参阅[更多信息](https://github.com/zyedidia/micro#dependencies)了解其他操作系统的操作。

### 手动安装

要手动安装 Micro，请下载最新的预编译二进制文件，静态 tar 压缩包，[参见此链接](https://github.com/zyedidia/micro/releases/)。从那里启动您喜欢的终端，`cd` 进入下载目录并解压：

```bash
tar xf micro_x.y.z-linux64.tar.gz
```

其中，`x.y.z` 是指版本号。

从那里，将 `micro` 移动到 `/usr/local/bin`，即您的路径中：

```bash
mv micro /usr/local/bin
```

## 配置 Micro

如果您从仓库安装的 micro，在进行任何更改之前，将默认设置复制到您的配置目录可能是一个好主意。要在当前用户下安装 `micro`，请使用以下命令：

```bash
micro -init
```

如果您希望为所有用户全局配置 `micro`，请将 `.micro` 隐藏文件夹复制到 `/etc/skel`：

```bash
cp -R .micro /etc/skel
```

并更改所有权和权限：

```bash
chown -R root:root /etc/skel/.micro
```

## Micro 快捷键备忘单

运行 `micro`：

```bash
micro
```

### 缓冲区(Buffer)选项卡

| 快捷键 | 快捷键 | 描述 |
|---|---|---|
| Ctrl+逗号 | Ctrl+, | 打开命令行接口(命令行模式) |
| Ctrl+Q | Ctrl+Q | 关闭 micro |
| Ctrl+O | Ctrl+O | 打开单个文件 |
| Ctrl+S | Ctrl+S | 保存文件 |
| Ctrl+W | Ctrl+W | 下一个选项卡 |
| Ctrl+Q | Ctrl+Q | 上一个选项卡/ 关闭 micro |
| Ctrl+Shift+T | Ctrl+Shift+T | 添加新选项卡 |
| Ctrl+Shift+逗号 | Ctrl+Shift+, | 显示按键绑定(按键绑定模式) |

### 移动

| 快捷键 | 快捷键 | 描述 |
|---|---|---|
| Ctrl+B | Ctrl+B | shell 模式 |
| Alt+G | Alt+G | 跳转到指定行 |
| Ctrl+Home | Ctrl+Home | 跳转到文件顶部 |
| Ctrl+End | Ctrl+End | 跳转到文件底部 |
| Ctrl+A | Ctrl+A | 移至行首 |
| Ctrl+E | Ctrl+E | 移至行尾 |
| Ctrl+L | Ctrl+L | 跳转行 |
| PgUp | PgUp | 向上滚动 |
| PgDn | PgDn | 向下滚动 |

## 结论

Micro 是您编辑文本文件时一个非常合适的工具。它比 `nano` 更强大，是 `vi`、`vim`、`neovim` 或 `emacs` 等重量级编辑器的优秀替代品。如果您希望拥有一个功能齐全、支持多种现代编辑功能的终端编辑器，Micro 是一个很好的选择。
