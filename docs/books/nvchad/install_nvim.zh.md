---
title: 安装 Neovim
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.1
tags:
  - nvchad
  - nvim
  - coding
---

# :simple-neovim: 安装 Neovim

## :material-message-outline: Neovim 简介

Neovim 因其速度、易定制性和配置而成为最好的代码编辑器之一。

Neovim 是 ==Vim== 编辑器的一个分支。它诞生于 2014 年，主要原因是当时 Vim 缺乏异步任务支持。用 ==Lua== 语言编写，目标是模块化代码以使其更易于管理，Neovim 是面向现代用户设计的。正如官方网站所述：

> Neovim 是为那些想要 Vim 最佳部分以及更多功能的用户构建的。

Neovim 的开发者选择 Lua，因为它非常适合嵌入，使用 LuaJIT 速度快，并且具有简单、面向脚本的语法。

从 0.5 版本开始，Neovim 包含了 ==Treesitter==（一个解析器生成器工具）并支持 ==Language Server Protocol==（LSP，语言服务器协议）。这减少了实现高级编辑功能所需的插件数量，提高了代码补全和 linting（代码检查）等操作的性能。

它的优势之一是定制性。所有配置都包含在一个文件中，可以通过版本控制系统（Git 或其他）分发到各种安装中，使其始终保持同步。

### :fontawesome-solid-users-gear: 开发者社区

虽然 Vim 和 Neovim 都是开源项目并托管在 GitHub 上，但开发模式存在显著差异。Neovim 拥有更加开放的社区开发模式，而 Vim 的开发更受其创作者选择的约束。与 Vim 相比，Neovim 的用户和开发者基础相当小，但它是一个持续增长的项目。

### :material-key: 关键特性

- 性能：非常快。
- 可定制：庞大的插件和主题生态系统
- 语法高亮：与 Treesitter 和 LSP 集成，但需要一些配置

与 Vim 一样，Neovim 需要基本了解其命令和选项。你可以通过 `:Tutor` 命令了解其功能概述，该命令调用一个文件，你可以在其中阅读并练习使用。学习时间比完全图形化的 IDE 更长，但一旦学会了命令的快捷方式和内置功能，你在编辑文档时将非常流畅。

![Nvim Tutor](./images/neovim_tutor.png)

## :material-monitor-arrow-down-variant: Neovim 安装

!!! warning "从 EPEL 安装"

    Neovim 也可以从 EPEL 仓库安装。可用的版本通常太旧，无法满足 NvChad 安装的最低要求。  
    强烈不鼓励通过此方法安装，本指南不支持此方法。

=== "从预编译包安装"

    使用预编译包可以安装满足要求的开发版和稳定版，并可用作配置 NvChad 的基础。

    要使用编辑器的全部功能，需要满足 Neovim 所需的依赖项，方法是手动提供预编译包的依赖项。所需软件包可以通过以下命令安装：

    ```bash
    dnf install compat-lua-libs libtermkey libtree-sitter libvterm luajit luajit2.1-luv msgpack unibilium xsel
    ```

    安装所需依赖项后，是时候获取选定的包了。

    访问[发布页面](https://github.com/neovim/neovim/releases)，可以下载开发版（==pre-release==）或稳定版（==stable==）。
    在这两种情况下，要为我们的架构下载的压缩文件是 ==linux64==。

    所需文件是 ==nvim-linux64.tar.gz==，我们也应该下载 ==nvim-linux64.tar.gz.sha256sum== 文件以验证其完整性。

    假设两者都下载到同一文件夹，我们将使用以下命令进行验证：

    ```bash
    sha256sum -c nvim-linux64.tar.gz.sha256sum
    nvim-linux64.tar.gz: OK
    ```
    
    现在将预编译包解压到你的主文件夹内的一个位置，在本指南中选择了 `.local/share/` 位置，但可以根据你的需要更改。运行命令：

    ```bash
    tar xvzf nvim-linux64.tar.gz -C ~/.local/share/
    ```

    此时剩下要做的就是为预编译包的 nvim 可执行文件在 ~/.local/bin/ 中创建一个符号链接。

    ```bash
    cd ~/.local/bin/
    ln -sf ~/.local/share/nvim-linux64/bin/nvim nvim
    ```

    要验证正确的安装，在终端中运行 `nvim -v` 命令，现在应该会显示类似以下内容：

    ```txt
    nvim -v
    NVIM v0.9.5
    Build type: RelWithDebInfo
    LuaJIT 2.1.1692716794
    ```

=== "从源代码安装"

    从预编译包安装仅为运行它的用户提供 `nvim`。如果你想使 Neovim 对系统的所有用户都可用，将不得不从源代码安装。编译 Neovim 并不特别困难，包括以下步骤。

    我们首先安装编译所需的软件包：

    ```bash
    dnf install --enablerepo=crb ninja-build libtool autoconf automake cmake gcc gcc-c++ make pkgconfig unzip patch gettext curl git
    ```

    一旦我们安装了必要的软件包，我们需要创建一个文件夹来在其中构建 neovim 并进入其中：

    Neovim 克隆默认与 Neovim 开发分支同步（在编写本文时，版本为 0.10.0）。要编译稳定版本，我们必须在克隆前切换到相应的分支：

    ```bash
    mkdir -p ~/lab/build
    cd ~/lab/build
    ```

    现在克隆仓库：

    ```bash
    git clone https://github.com/neovim/neovim
    ```

    操作完成后，我们将有一个名为 *neovim* 的文件夹，包含所有必要文件。下一步是检出稳定分支，然后使用 `make` 命令配置和编译源代码。

    ```bash
    cd ~/lab/build/neovim/
    git checkout stable
    make CMAKE_BUILD_TYPE=RelWithDebInfo
    ```

    我们选择 `RelWithDebInfo` 类型，因为它提供优化，以及一个有用的调试层，用于后续定制。如果你更喜欢最大性能，也可以使用 `Release` 类型。

    该过程负责配置和编译要放入我们系统的文件。这些文件保存在 `neovim/build` 中。要安装它们，我们将使用 *make install* 命令：

    ```bash
    make install
    ```

    由于此命令将修改文件系统，必须以超级用户身份运行，可以使用 `sudo` 或直接以 root 用户身份。

    安装完成后，我们可以通过检查 Neovim 的路径来验证一切顺利：

    ```bash
    whereis nvim
    nvim: /usr/local/bin/nvim
    ```

    并验证版本：

    ```bash
    nvim --version
    NVIM v0.9.5
    Build type: Release
    LuaJIT 2.1.1692716794
    ....
    ```

    从上面的命令摘录中可以看出，这里执行了稳定版的安装。稳定版和开发版在 Rocky Linux 9 上都可以与 NvChad 完美配合。

    ### :material-package-variant-closed-remove: 卸载

    如果我们需要删除安装，例如切换到另一个版本，我们必须将自己带回构建文件夹，并使用 Neovim 提供的 cmake `target`。要执行卸载，需要执行以下命令：

    ```bash
    cmake --build build/ --target uninstall
    ```

    该命令也需要超级用户权限或以 *root* 用户身份运行。

    或者，你可以使用手动方法，通过以下方式删除可执行文件和库：

    ```bash
    rm /usr/local/bin/nvim
    rm -r /usr/local/share/nvim/
    ```

    同样，你需要以超级用户权限执行这些命令。

## :material-image-outline: Neovim 基础

从截图中可以看到，Neovim 的基本安装提供了一个尚不能与 IDE 相提并论的编辑器。

![Neovim Standard](./images/nvim_standard.png)

现在我们有了基本的编辑器，是时候通过 [NvChad](install_nvchad.md) 提供的配置将其改造成更高级的工具了。
