---
title: 安装 NvChad
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.1
tags:
  - nvchad
  - coding
  - editor
---

# :simple-neovim: 将 Neovim 变成高级 IDE

## :material-arrow-bottom-right-bold-outline: 前提条件

正如 NvChad 网站上所述，你需要确保系统满足以下要求：

* [Neovim 0.10.0](https://github.com/neovim/neovim/releases/tag/v0.10.0)。
* [Nerd Font](https://www.nerdfonts.com/) 在你的终端模拟器中设置它。
    * 确保你设置的 nerd font 不以 **Mono** 结尾
    * **示例：** Iosevka Nerd Font 而不是 ~~Iosevka Nerd Font Mono~~
* [Ripgrep](https://github.com/BurntSushi/ripgrep) 需要通过 Telescope 进行 grep 搜索（**可选**）。
* GCC 和 Make

??? warning "执行全新安装"

    如要求中所述，在先前配置之上安装此新配置可能会造成无法修复的问题。建议进行全新安装。

### :material-content-save-cog-outline: 预备操作

如果你之前使用过 Neovim 安装，它将创建三个文件夹来写入文件，分别是：

```text
~/.config/nvim
~/.local/share/nvim
~/.cache/nvim
```

要执行配置的全新安装，我们需要首先备份先前的配置：

```bash
mkdir ~/backup_nvim
cp -r ~/.config/nvim ~/backup_nvim
cp -r ~/.local/share/nvim ~/backup_nvim
cp -r ~/.cache/nvim ~/backup_nvim
```

然后我们删除所有先前的配置和文件：

```bash
rm -rf ~/.config/nvim
rm -rf ~/.local/share/nvim
rm -rf ~/.cache/nvim
```

## :material-monitor-arrow-down-variant: 安装

配置结构的创建是通过使用 *Git* 从初始化仓库（==starter==）复制文件来实现的。该方法允许安装 NvChad 配置，该配置作为 Neovim 插件在 *lazy.nvim* 插件管理器中准备。  
这样，配置像所有其他插件一样更新，简化了用户的管理。此外，这种方法使得整个用户配置独立，允许其在不同机器间进行总体管理和分发。

要下载和初始化配置，使用以下命令：

```bash
git clone https://github.com/NvChad/starter ~/.config/nvim && nvim
```

该命令由两部分组成。第一部分将 *starter* 仓库的内容下载到 `~/.config/nvim/`（Neovim 设置的默认文件夹），而第二部分调用 ==nvim== 可执行文件，该可执行文件使用你刚下载的配置初始化编辑器。一旦你完成安装插件和解析器，你将面对以下屏幕。要关闭插件管理器，按 ++"q"++：

![NvChad Install](images/install_nvchad_25.png)

初始配置是最小的，为你的定制提供了一个起点。如截图所示，当编辑器首次启动时，只有四个模块（==plugins==），用复选标记标记，被加载，它们如下：

* **base46** - 提供编辑器主题
* **NvChad** - 允许用户配置进入 Neovim 的基本配置
* **nvim-treesitter** - 用于分析和高亮代码
* **ui** - 编辑器界面（statusline、tabufline..）

剩余的模块将在功能被请求时通过 ==*lazyloading*== 技术激活。这提高了编辑器的整体性能，尤其是加快了其启动时间。

此时，编辑器已准备就绪。以下各节深入介绍了安装过程，日常使用并非必需。如果你只对使用感兴趣，可以转到[使用 NvChad](./nvchad_ui/using_nvchad.md)页面。  
不过，仍然建议阅读[官方文档](https://nvchad.com/docs/quickstart/install)以了解其组件和功能。

要关闭编辑器，使用按键 ++colon++ ++"q "++。

### :material-timer-cog-outline: 引导过程

引导过程在 *starter* 仓库的 ==*init.lua*== 文件中实现，由以下步骤组成：

对默认主题路径和 `<leader>` 键的初始设置，这里使用 ++space++ 键：

```lua
vim.g.base46_cache = vim.fn.stdpath "data" .. "/nvchad/base46/"
vim.g.mapleader = " "
```

随后安装主要的 **lazy.nvim** 插件：

```lua
-- bootstrap lazy and all plugins
local lazypath = vim.fn.stdpath "data" .. "/lazy/lazy.nvim"

if not vim.loop.fs_stat(lazypath) then
  local repo = "https://github.com/folke/lazy.nvim.git"
  vim.fn.system { "git", "clone", "--filter=blob:none", repo, "--branch=stable", lazypath }
end

vim.opt.rtp:prepend(lazypath)

local lazy_config = require "configs.lazy"
```

以及安装 NvChad 插件和所有在 `plugins` 文件夹中配置的插件：

```lua
-- load plugins
require("lazy").setup({
  {
    "NvChad/NvChad",
    lazy = false,
    branch = "v2.5",
    import = "nvchad.plugins",
    config = function()
      require "options"
    end,
  },

  { import = "plugins" },
}, lazy_config)
```

然后将主题应用到 *default* 和 *statusline* 设置：

```lua
-- load theme
dofile(vim.g.base46_cache .. "defaults")
dofile(vim.g.base46_cache .. "statusline")
```

完成后，配置操作所需的 ==autocmds==（[Neovim 自动命令](https://neovim.io/doc/user/autocmd.html)）和键盘映射也被输入：

```lua
require "nvchad.autocmds"

vim.schedule(function()
  require "mappings"
end)
```

## :material-file-tree-outline: 配置结构

NvChad 安装的结构如下：

```text
~/.config/nvim/
├── init.lua
├── lazy-lock.json
├── LICENSE
├── lua
│   ├── chadrc.lua
│   ├── configs
│   │   ├── conform.lua
│   │   └── lazy.lua
│   ├── mappings.lua
│   ├── options.lua
│   └── plugins
│       └── init.lua
└── README.md
```

它由一个起始文件 **init.lua** 组成，该文件初始化并协调将定制插入到 ==Neovim== 的配置中，此文件最初看起来与上面显示的 **starter** 仓库中用于 *bootstrap* 的文件相同，稍后将用于将其他文件加载到配置中，例如自己的 *autocommands.lua* 文件。

接下来是 **lazy-lock.json** 文件，其中存储了安装中的所有插件及其在 *GitHub* 上相对于开发的状态。此文件允许在多台机器上的安装之间同步编辑器状态，并允许自定义安装复制所需的状态。

其余配置位于 `lua` 文件夹中，并从 **chadrc.lua** 文件开始初始化，该文件在初始版本中仅包含编辑器主题设置。  
此文件用于自定义编辑器的外观（==UI==），并与 **NvChad** 插件的 [nvconfig.lua](https://github.com/NvChad/NvChad/blob/v2.5/lua/nvconfig.lua) 文件共享语法；要编译它，只需将 *nvconfig.lua* 文件的所需部分复制到你自己的 *chadrc.lua* 中，并根据需要更改其属性。

配置使用的下一个文件（文件夹将在后面描述）是 **option.lua** 文件，用于编辑器自定义，例如缩进空格、与宿主系统共享剪贴板，以及非常重要的是，将 *Mason* 安装的二进制文件包含在路径中。  
与前一个文件一样，它与 **NvChad** 插件的[相应文件](https://github.com/NvChad/NvChad/blob/v2.5/lua/nvchad/options.lua)共享语法；要像上面一样自定义它，只需复制选项并编辑它们。

最后遇到的是 **mapping.lua** 文件，其中设置了用于调用编辑器提供的各种功能的键盘按键。初始文件包含进入 **COMMAND** 模式的按键映射、使用 *conform.nvim* 进行格式化的按键以及退出 **INSERT** 模式的按键。  
这些按键使用 Neovim 原生的 `vim.keymap.set` 语法，对于它们的配置，你可以参考 NvChad 的[默认映射](https://github.com/NvChad/NvChad/blob/v2.5/lua/nvchad/mappings.lua)，或者参考 Neovim 中包含的帮助页面 `:h vim.keymap.set`。

```lua
require "nvchad.mappings"

-- add yours here

local map = vim.keymap.set

map("n", ";", ":", { desc = "CMD enter command mode" })

map("n", "<leader>fm", function()
  require("conform").format()
end, { desc = "File Format with conform" })

map("i", "jk", "<ESC>", { desc = "Escape insert mode" })
```

配置中包含的两个文件夹 `configs` 和 `plugins` 都用于管理插件；个人插件应放在 `plugins` 文件夹中，它们的额外配置（如果有）放在 `configs` 文件夹中。  
最初将有一个 *plugins/init.lua* 文件，其中安装了在 *configs/conform.lua* 中配置的 *conform.lua* 插件和 *nvimtree.nvim*，其中包含与 *Git* 相关的装饰选项。

!!! notes "插件的组织"

    插件的包含是通过插入 `plugins` 文件夹中存在的任何经过适当配置的文件来完成的，这允许按目的（例如）来组织插件，通过创建单独的文件（*utils.lua*、*editor.lua*、*markdown.lua* 等），这样就可以更有条理地处理配置。

还有与 *licensing* 相关的文件和一个从 **starter** 仓库复制的 *README.md*，可用于在配置在 *Git* 仓库中维护时说明你的配置。

## :material-keyboard-outline: 主要键盘按键

这是返回基本命令映射的调用：

```lua
vim.schedule(function()
  require "mappings"
end)
```

它设置了四个主键，与这些键关联的其他键可以启动命令。主键是：

* C = ++ctrl++
* leader = ++space++
* A = ++alt++
* S = ++shift++

!!! note

    在这些文档中，我们将多次引用这些键映射。

默认映射包含在 NvChad 插件的 *lua/mapping.lua* 中，但可以使用你自己的 *mappings.lua* 扩展其他自定义命令。

`<leader>th` 更换主题 ++space++ + ++"t"++ + ++"h"++  
`<C-n>` 打开 nvimtree ++ctrl++ + ++"n"++  
`<A-i>` 在浮动标签中打开终端 ++alt++ + ++"i"++

有许多为你预设的组合，涵盖了 NvChad 的所有使用场景。在开始使用配置了 NvChad 的 Neovim 实例之前，值得花时间分析一下按键映射。
