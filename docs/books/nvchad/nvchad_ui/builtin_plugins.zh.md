---
title: 内置插件
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.8, 9.3
tags:
    - nvchad
    - coding
    - plugins
---

# :material-folder-multiple-outline: 基本配置插件

!!! note "插件命名约定"

    本章将使用 `user_github/plugin_name` 格式标识插件。这是为了避免与类似名称的插件可能出现的错误，并介绍 NvChad 和 `custom` 配置用于插件条目的格式。

基本的 NvChad 插件位于 `~/.local/share/nvim/lazy/NvChad/lua/nvchad/plugins/` 文件夹中：

```text title=".local/share/nvchad/lazy/NvChad/lua/nvchad/plugins/"
├── init.lua
└── ui.lua
```

各自的配置在 `~/.local/share/nvim/lazy/NvChad/lua/nvchad/configs/` 文件夹中：

```text title=".local/share/nvchad/lazy/NvChad/lua/nvchad/configs/"
├── cmp.lua
├── gitsigns.lua
├── lazy_nvim.lua
├── lspconfig.lua
├── luasnip.lua
├── mason.lua
├── nvimtree.lua
├── telescope.lua
└── treesitter.lua
```

在 `plugins` 文件夹中，有 *init.lua* 和 *ui.lua* 文件。前者处理为编辑器提供附加功能的插件配置（*telescope*、*gitsigns*、*tree-sitter* 等），而后者设置编辑器的外观（颜色、图标、文件管理器等）。
  
## :material-download-circle-outline: 基本插件

以下是主要插件的简要分析：

=== "init.lua 插件"

    - [nvim-lua/plenary.nvim](https://github.com/nvim-lua/plenary.nvim) - 提供一个常用 lua 函数库，供其他插件使用，例如 *telescope* 和 *gitsigns*。

    - [stevearc/conform.nvim](https://github.com/stevearc/conform.nvim) 为 Neovim 提供的格式化插件，快速且可扩展，得益于用户配置提供的 `configs/conform.lua` 文件

    - [nvim-treesitter/nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter) - 允许你在 Neovim 中使用 tree-sitter 接口，并提供一些基本功能，如高亮。

    - [lewis6991/gitsigns.nvim](https://github.com/lewis6991/gitsigns.nvim) - 为 *git* 提供装饰，报告添加、删除和更改的行——这些报告也集成到 *statusline* 中。

    - [williamboman/mason.nvim](https://github.com/williamboman/mason.nvim) - 通过便捷的图形界面简化了对 LSP（语言服务器）安装的管理。

    - [neovim/nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) - 提供几乎所有可用语言服务器的适当配置。它是一个社区集合，已经设置了最相关的设置。该插件负责接收我们的配置并将它们放入编辑器环境中。

    - [hrsh7th/nvim-cmp](https://github.com/hrsh7th/nvim-cmp) 及其各个源，由以下插件提供：

        - [L3MON4D3/LuaSnip](https://github.com/L3MON4D3/LuaSnip)
        - [saadparwaiz1/cmp_luasnip](https://github.com/saadparwaiz1/cmp_luasnip)
        - [hrsh7th/cmp-nvim-lua](https://github.com/hrsh7th/cmp-nvim-lua)
        - [hrsh7th/cmp-nvim-lsp](https://github.com/hrsh7th/cmp-nvim-lsp)
        - [hrsh7th/cmp-buffer](https://github.com/hrsh7th/cmp-buffer)
        - [hrsh7th/cmp-path](https://github.com/hrsh7th/cmp-path)

    - [windwp/nvim-autopairs](https://github.com/windwp/nvim-autopairs) - 通过此插件，我们拥有了括号和其他字符的自动关闭功能。例如，通过插入一个开始括号 `(` 完成将自动插入关闭括号 `)` 并将光标放在中间。

    - [numToStr/Comment.nvim](https://github.com/numToStr/Comment.nvim) - 提供代码注释的高级功能。

    - [nvim-telescope/telescope.nvim](https://github.com/nvim-telescope/telescope.nvim) - 提供高级文件搜索功能，高度可定制，也可（例如）用于选择 NvChad 主题（命令 `:Telescope themes`）。

    ![Telescope find_files](../images/telescope_find_files.png)

=== "ui.lua 插件"

    - [NvChad/base46](https://github.com/NvChad/base46) - 提供界面主题。

    - [NvChad/ui](https://github.com/NvChad/ui) - 提供实际界面和 NvChad 的核心工具。通过此插件，我们可以有一个在编辑期间提供信息的 *statusline* 和一个允许我们管理打开缓冲区的 *tabufline*。此插件还提供工具 **NvChadUpdate** 用于更新，**NvCheatsheet** 用于键盘快捷键概览，以及 **Nvdash** 用于文件操作。

    - [NvChad/nvim-colorizer.lua](https://github.com/NvChad/nvim-colorizer.lua) - 另一个由 NvChad 开发者编写的插件。它专门是一个高性能的颜色高亮器。

    - [kyazdani42/nvim-web-devicons](https://github.com/kyazdani42/nvim-web-devicons)—这为 IDE 中的文件类型和文件夹添加图标（需要使用一个 Nerd Font）。这使我们能够在文件资源管理器中视觉识别文件类型，加快操作速度。

    - [lukas-reineke/indent-blankline.nvim](https://github.com/lukas-reineke/indent-blankline.nvim) - 提供引导线以更好地识别文档中的缩进，使子程序和嵌套命令易于识别。

    - [kyazdani42/nvim-tree.lua](https://github.com/kyazdani42/nvim-tree.lua) - Neovim 的文件资源管理器，允许最常见的文件操作（复制、粘贴等），具有 Git 集成，用不同的图标标识文件，以及其他功能。最重要的是，它会自动更新（这在处理 Git 仓库时非常有用）。

    ![Nvim Tree](../images/nvim_tree.png)

    - [folke/which-key.nvim](https://github.com/folke/which-key.nvim) - 显示输入部分命令时所有可能的自动完成选项。

    ![Which Key](../images/which_key.png)

## 总结与最终想法

NvChad 开发者们投入了大量的工作，这一点必须得到认可。他们在所有插件之间创建了一个集成的环境，使用户界面干净且专业。此外，*在幕后*工作的插件实现了增强的编辑和其他功能。

这意味着普通用户可以立即拥有一个基本的 IDE 来开始工作，以及一个可以根据他们的需求进行调整的可扩展配置。
