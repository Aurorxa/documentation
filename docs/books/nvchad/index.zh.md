---
title: 概述
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.1
tags:
  - nvchad
  - coding
  - editor
---

# :material-book-open-page-variant-outline: 简介

在本手册中，你将找到实现 Neovim 以及 NvChad 的方法，以创建功能齐全的 ==**I**ntegrated **D**evelopment **E**nvironment==（IDE，集成开发环境）。

我说"方法"是因为有很多可能性。作者在这里侧重于使用这些工具来编写 markdown，但如果 markdown 不是你的重点，不用担心，继续阅读即可。如果你对这些工具（NvChad 或 Neovim）不熟悉，那么本手册将为你介绍这两者，并且如果你逐步完成这些文档，你将很快意识到你可以将此环境设置为对你的任何编程或脚本编写需求都有很大帮助的工具。

想要一个能帮助编写 Ansible playbook 的 IDE 吗？你可以实现！想要一个适用于 Golang 的 IDE 吗？那也是可用的。只是想要一个编写 BASH 脚本的良好界面吗？它也可用。

通过使用 ==**L**anguage **S**erver **P**rotocols==（LSP，语言服务器协议）和 ==linters==（代码检查工具），你可以设置一个专为你定制的环境。最棒的是，一旦环境设置好，通过使用 [lazy.nvim](https://github.com/folke/lazy.nvim) 和 [Mason](https://github.com/williamboman/mason.nvim)，当有新的更改可用时可以快速更新，这两者在这里都会涉及。

由于 Neovim 是 [Vim](https://www.vim.org/) 的一个分支，整体界面对 *vim* 用户来说会很熟悉。如果你不是 *vim* 用户，使用本手册可以很快掌握命令语法。这里涵盖了很多信息，但很容易跟着操作，一旦完成内容，你将充分了解如何使用这些工具为你*自己的*需求构建自己的 IDE。

作者的本意是**不**将本手册分解为章节。原因是这意味着必须遵循某种顺序，而在大多数情况下，这并不必要。你*会*想从本页面开始，阅读并遵循"额外软件"、"安装 Neovim"和"安装 NvChad"部分，但从那里起，你可以选择如何继续。

## :simple-neovim: 将 Neovim 用作 IDE

Neovim 的基本安装为开发提供了一个出色的编辑器，但它还不能被称为 IDE；所有更高级的 IDE 功能，即使已经预设，但尚未激活。要做到这一点，我们需要将必要的配置传递给 Neovim，这就是 NvChad 来帮助我们解决的问题。这使我们只需一个命令就能获得开箱即用的基本配置！

配置是用 Lua 编写的，Lua 是一种非常快的编程语言，使得 NvChad 的启动、命令和按键的执行时间都非常快。这也是通过用于插件的 ==Lazy loading==（延迟加载）技术实现的，该技术仅在需要时才加载它们。

界面非常干净和舒适。

正如 NvChad 开发者所强调的，该项目只是作为构建你自己个人 IDE 的基础。后续的定制通过使用插件来完成。

![NvChad UI](images/nvchad_rocky.png)

### 主要功能

* :material-run-fast: **为速度而设计。** 从编程语言的选择到加载组件的技术，一切都是为了最小化执行时间。
* :material-invert-colors: **美观的界面。** 尽管是一个 *cli* 应用程序，但界面看起来现代且图形美观，所有组件与 UI 完美契合。
* :material-file-settings-outline: **高度可配置。** 由于基础应用程序（NeoVim）的模块化，编辑器可以完美地适应个人需求。然而，请记住，当我们谈论定制时，我们指的是功能，而非界面的外观。
* :material-update: **自动更新机制。** 编辑器带有一个机制（通过使用 *git*），允许通过简单的 `:NvChadUpdate` 命令进行更新。
* :material-language-lua: **由 Lua 驱动。** NvChad 的配置完全用 *lua* 编写，这使其能够无缝集成到 Neovim 的配置中，充分利用其所基于的编辑器的全部潜力。
* :material-palette-outline: **众多内置主题。** 该配置已经包含大量可用主题，始终记住我们讨论的是一个 *cli* 应用程序，主题可以通过 `<leader> + th` 键选择。

![NvChad Themes](images/nvchad_th.png)

## 参考资料

### :simple-lua: Lua

#### 什么是 Lua？

Lua 是一种健壮、轻量级的脚本语言，支持多种编程方法。"Lua" 这个名字来自葡萄牙语，意思是"月亮"。

Lua 由里约热内卢天主教大学的 Roberto Ierusalimschy、Luiz Henrique de Figueiredo 和 Waldemar Celes 开发。由于直到 1992 年巴西都受到对硬件和软件严格进口法规的约束，因此出于纯粹的必要性，这三位程序员开发了自己的脚本语言，称为 Lua。

由于 Lua 主要专注于脚本，因此很少作为独立的编程语言使用。相反，它最常被用作可以集成（嵌入）到其他程序中的脚本语言。

Lua 用于视频游戏和游戏引擎（Roblox、Warframe..）的开发，作为许多网络程序（Nmap、ModSecurity..）中的编程语言，以及作为工业程序中的编程语言。Lua 也作为一个库，开发者可以将其集成到他们的程序中，通过仅作为宿主应用程序的组成部分来启用脚本功能。

#### Lua 的工作原理

Lua 有两个主要组件：

* Lua 解释器
* Lua 虚拟机（VM）

Lua 不像其他语言（例如 Python）那样通过 Lua 文件直接解释。相反，它使用 Lua 解释器将 Lua 文件编译为字节码。Lua 解释器高度可移植，能够在多种设备上运行。

#### 关键特性

* 速度：Lua 被认为是最快的编程语言之一，属于解释型脚本语言；它可以比大多数其他编程语言更快地执行非常耗费性能的任务。
* 大小：与其他编程语言相比，Lua 非常小。这种小尺寸非常适合将 Lua 集成到多个平台，从嵌入式设备到游戏引擎。
* 可移植性和集成性：Lua 的可移植性几乎无限。任何支持标准 C 编译器的平台都可以毫无问题地运行 Lua。Lua 不需要复杂的重写来与其他编程语言兼容。
* 简单性：Lua 设计简单但提供强大的功能。Lua 的主要特性之一是元机制，允许开发者实现自己的功能。语法简单易懂，任何人都可以学习 Lua 并在自己的程序中使用它。
* 许可证：Lua 是自由开源软件，基于 MIT 许可证分发。这允许任何人将其用于任何目的，无需支付任何许可证费用或版税。

### :simple-neovim: Neovim

Neovim 在其[专属页面](./install_nvim.md)中有详细描述，因此我们只讨论其主要特性：

* 性能：非常快。
* 可定制：庞大的插件和主题生态系统。
* 语法高亮：与 Treesitter 和 LSP 集成（需要一些额外配置）。
* 跨平台：Linux、Windows 和 macOS
* 许可证：MIT：简短简单的宽松许可证，条件仅要求保留版权和许可声明。

### :material-protocol: LSP

什么是 **L**anguage **S**erver **P**rotocol（语言服务器协议）？

语言服务器是一个标准化的语言库，它使用自己的程序（协议）为诸如自动完成、转到定义或鼠标悬停定义等功能提供支持。

Language Server Protocol（LSP，语言服务器协议）背后的想法是将工具和服务器之间的通信协议标准化，这样单个语言服务器就可以在多个开发工具中复用。这样，开发者可以简单地将这些库集成到他们的编辑器中并引用现有的语言基础设施，而不是定制他们的代码来包含它们。

### :material-file-document-check-outline: tree-sitter

[Tree-sitter](https://tree-sitter.github.io/tree-sitter/) 基本上由两个组件组成：一个 ==parser generator==（解析器生成器）和一个 ==incremental parsing library==（增量解析库）。它可以构建源文件的语法树，并在每次更改时高效地更新它。

解析器是一个将数据分解为更小元素的组件，以便于将其翻译成另一种语言，或者在我们的情况下，然后传递给解析库。一旦源文件被分解，解析库解析代码并将其转换为语法树，从而更智能地操作代码结构。这使得改进（并加速）成为可能：

* 语法高亮
* 代码导航
* 重构
* 文本对象和移动

??? notes "LSP 和 tree-sitter 的互补性"

    虽然看起来这两个服务（LSP 和 tree-sitter）是冗余的，但实际上它们是互补的，因为 LSP 在项目级别工作，而 tree-sitter 仅在打开的源文件上工作。

现在我们已经解释了一些用于创建 IDE 的技术，我们可以继续配置 NvChad 所需的[额外软件](./additional_software.md)。
