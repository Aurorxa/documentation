---
title: 额外软件
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.1
tags:
  - nvchad
  - coding
---

# :material-cart-plus: 所需额外软件

虽然并非必需，但有几个额外的软件将有助于 NvChad 的整体使用。以下各节将引导你了解这些软件及其用途。

## :material-text-search: RipGrep

`ripgrep` 是一种面向行的搜索工具，它递归搜索当前目录中与 *regex*（正则表达式）模式匹配的内容。默认情况下，*ripgrep* 遵循 *gitignore* 的规则并自动跳过隐藏文件/目录和二进制文件。Ripgrep 在 Windows、macOS 和 Linux 上提供出色的支持，每个版本都有可用的二进制文件。

=== "从 EPEL 安装 RipGrep"

    在 Rocky Linux 8 和 9 中，你可以从 EPEL 安装 RipGrep。为此，安装 `epel-release`，升级系统，然后安装 `ripgrep`：

    ```bash
    sudo dnf install -y epel-release
    sudo dnf upgrade
    sudo dnf install ripgrep
    ```

=== "使用 cargo 安装 RipGrep"

    Ripgrep 是用 *Rust* 编写的软件，可以使用 `cargo` 工具进行安装。但请注意，`cargo` 不是 *rust* 默认安装的一部分，因此你必须显式安装它。如果你使用此方法时遇到错误，请回退到从 EPEL 安装。

    ```bash
    dnf install rust cargo
    ```

    一旦安装了必要的软件，我们可以安装 `ripgrep`：

    ```bash
    cargo install ripgrep
    ```

    安装将把 `rg` 可执行文件保存在 `~/.cargo/bin` 文件夹中，该文件夹不在 PATH 中，为了在用户级别使用它，我们将它链接到 `~/.local/bin/`。

    ```bash
    ln -s ~/.cargo/bin/rg ~/.local/bin/
    ```

## :material-check-all: RipGrep 验证

此时我们可以通过以下命令检查一切是否正常：

```bash
rg --version
ripgrep 13.0.0
-SIMD -AVX (compiled)
+SIMD +AVX (runtime)
```

RipGrep 需要通过 `:Telescope` 进行递归搜索。

## :material-source-merge: Lazygit

[LazyGit](https://github.com/jesseduffield/lazygit) 是一个 ncurses 风格的界面，允许你以更用户友好的方式执行所有 `git` 操作。它是 ==lazygit.nvim== 插件所必需的。此插件使得可以直接从 NvChad 使用 LazyGit，它打开一个浮动窗口，你可以从中对仓库执行所有操作，从而无需离开编辑器即可对 *git 仓库* 进行所有更改。

要安装它，我们可以使用 Fedora 的仓库。在 Rocky Linux 9 上它可以完美运行。

```bash
sudo dnf copr enable atim/lazygit -y
sudo dnf install lazygit
```

安装后，我们打开一个终端并输入 `lazygit` 命令，将出现类似以下界面：

![LazyGit UI](./images/lazygit_ui.png)

使用 ++"?"++ 键，我们可以调出包含所有可用命令的菜单。

![LazyGit UI](images/lazygit_menu.png)

现在我们的系统上已经有了所有必要的支持软件，我们可以继续安装基本软件。我们将从整个配置所基于的编辑器开始，[Neovim](install_nvim.md)。
