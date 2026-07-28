---
title: 示例配置
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.1
tags:
  - nvchad
  - coding
  - plugins
---

# 示例配置

!!! danger "不再提供"

    安装 NvChad 时不再提供示例配置，因此此页面已过时，将在新版指南中删除。相关说明将尽快更新。

## :material-message-outline: 简介

NvChad 2.0 版本引入了在安装阶段创建 ==custom== 文件夹的功能。该文件夹的创建是通过修改其文件来自定义编辑器的起点。它在启动时安装，允许在首次启动时获得具有 IDE（集成开发环境）基本功能的编辑器，但也可以在安装 NvChad 后引入。

其安装最重要的方面是为引入语言服务器、linters 和格式化程序等高级功能创建基本结构。这些结构只需少量修改即可集成必要功能。

该文件夹是从 NvChad GitHub 仓库中的示例文件夹创建的：([example-config](https://github.com/NvChad/example_config))。

## :material-monitor-arrow-down-variant: 安装

=== "在启动时安装"

    要在安装期间创建它，对安装开始时询问的问题回答 "y"：

    > 是否要安装示例自定义配置？(y/N)：

    肯定回答将启动一个过程，将 *example-config* 文件夹的内容从 GitHub 克隆到 **~/.config/nvim/lua/custom/**，完成后将从中删除 **.git** 文件夹。  
    删除它允许该文件夹置于个人版本控制之下。

    文件夹准备就绪，下次启动 NvChad 时将用于向编辑器输入自定义配置。

=== "从仓库安装"

    ==example-config== 提供的配置安装也可以在安装 NvChad 后进行，在这种情况下，仍然使用仓库，但通过手动操作获取。

    不带 ==example-config== 的标准安装仍然会创建一个 *custom* 文件夹，用于保存 ==chadrc.lua== 文件以进行用户自定义，应将其删除或保存到 ==backup== 中，以允许克隆运行。然后使用以下命令保存现有配置：

    ```bash
    mv ~/.config/nvim/lua/custom/ ~/.config/nvim/lua/custom.bak
    ```

    然后将 GitHub 仓库克隆到你的配置中：

    ```bash
    git clone https://github.com/NvChad/example_config.git ~/.config/nvim/lua/custom
    ```

    该命令将在线仓库的完整内容复制到 `~/.config/nvim/lua/custom/` 文件夹中，包括隐藏的 `.git` 文件夹，你必须手动删除它才能切换到个人版本控制。使用以下命令删除它：

    ```bash
    rm rf ~/.config/nvim/lua/custom/.git/
    ```

    文件夹准备就绪，下次启动 NvChad 时将用于向编辑器输入自定义配置。

## :material-file-outline: 结构

==custom== 文件夹的结构由多个配置文件和一个包含 *plugins.lua* 中设置的插件选项文件的 `configs` 文件夹组成。

为插件设置使用单独的文件，可以拥有一个更精简的 *plugins.lua* 文件，并且在自定义插件时只需处理插件代码。这也是开发后续添加插件的推荐方法。

创建的结构如下：

```text
custom/
├── chadrc.lua
├── configs
│   ├── conform.lua
│   ├── lspconfig.lua
│   └── overrides.lua
├── highlights.lua
├── init.lua
├── mappings.lua
├── plugins.lua
└── README.md

```

如你所见，该文件夹包含一些与 NvChad 基本结构中的文件同名的文件。这些文件允许你集成配置并覆盖编辑器的基本设置。

## :octicons-file-code-16: 结构分析

现在让我们来检查其内容：

### :material-file-multiple-outline: 主要文件

#### :material-language-lua: chadrc.lua

```lua
---@type ChadrcConfig
local M = {}

-- 覆盖主题和高亮文件的路径
local highlights = require "custom.highlights"

M.ui = {
  theme = "onedark",
  theme_toggle = { "onedark", "one_light" },

  hl_override = highlights.override,
  hl_add = highlights.add,
}

M.plugins = "custom.plugins"

-- 检查 core.mappings 获取表结构
M.mappings = require "custom.mappings"

return M
```

该文件由 **~/.config/nvim/lua/core/utils.lua** 文件中设置的 `load_config` 函数插入到 Neovim 配置中。该函数负责加载默认设置，如果存在，也加载 *custom* 文件夹中 *chadrc.lua* 文件的设置：

```lua
M.load_config = function()
  local config = require "core.default_config"
  local chadrc_path = vim.api.nvim_get_runtime_file("lua/custom/chadrc.lua", false)[1]
...
```

其功能是将 *custom* 文件夹中的文件插入到 NvChad 配置中，然后将它们与默认文件一起用于启动 *Neovim* 实例。这些文件通过 `require` 函数插入到配置树中，例如：

```lua
require("custom.mappings")
```

字符串 **custom.mappings** 表示文件的不含扩展名的相对路径，与默认路径相对，此时的默认路径是 **~/.config/nvim/lua/**。点号替代了斜杠，因为这是用 Lua 编写的代码中的惯例（*lua 语言*中没有 *目录* 的概念）。

概括来说，上述调用将 `custom/mappings.lua` 文件中编写的配置插入到 NvChad 映射中，从而插入用于调用 `custom/plugins.lua` 中设置的插件命令的快捷键。

文件中还有一个部分覆盖了 `core/default_config.lua` 中包含的 NvChad 用户界面配置的某些设置，特别是 **M.ui** 部分，它允许你选择明暗主题。

文件末尾设置了指向 `custom/plugins.lua` 文件的 ==require== 调用，对应字符串：

```lua
M.plugins = "custom.plugins"
```

这样，`custom/plugins.lua` 中设置的插件将与构成 NvChad 配置的插件一起传递给 *lazy.nvim* 进行安装和管理。在这种情况下，包含不是在 Neovim 树中，而是在 *lazy.nvim* 的配置中，因为此插件通过调用 `vim.go.loadplugins = false` 完全禁用了相关的编辑器功能。

#### :material-language-lua: init.lua

此文件用于覆盖 `core/init.lua` 中定义的设置，例如缩进或写入磁盘的交换间隔。它还用于创建自动命令，如文件中的注释行所述。示例如下，其中包含了一些用于编写 Markdown 文档的设置：

```lua
--local autocmd = vim.api.nvim_create_autocmd

-- Markdown 设置
local opt = vim.opt

opt.tabstop = 4
opt.softtabstop = 4
opt.shiftwidth = 4
opt.shiftround = false
opt.expandtab = true
opt.autoindent = true
opt.smartindent = true

-- 调整 nvim 窗口大小时自动调整窗格大小
--autocmd("VimResized", {
--   pattern = "*",
--   command = "tabdo wincmd =",
-- })
```

这将 2 空格制表符替换为更适合 Markdown 代码的 4 空格制表符。

#### :material-language-lua: plugins.lua

此文件设置要添加到基本 NvChad 配置中的插件。添加插件在[插件管理器](nvchad_ui/plugins_manager.md)专用页面中有详细说明。

*example-config* 创建的 *plugins.lua* 文件在第一部分中包含了若干自定义设置，这些设置覆盖了插件定义选项和默认插件配置。这部分文件我们不需要修改，因为开发者已在 *config* 文件夹中准备了用于此目的的专门文件。

接下来是一个插件的安装示例，创建此示例是为了让你熟悉 *lazy.nvim* 使用的格式。

```lua
  -- 安装一个插件
  {
    "max397574/better-escape.nvim",
    event = "InsertEnter",
    config = function()
      require("better_escape").setup()
    end,
  },
```

你可以在此插件之后和最后一个括号之前插入所有附加插件。有一个适用于各种用途的完整插件生态。你可以访问 [Neovimcraft](https://neovimcraft.com/) 获取初步概览。

#### :material-language-lua: mappings.lua

此文件用于将调用附加插件命令所需的映射（键盘快捷键）包含到配置树中。

这里也展示了一个示例设置，以便研究其格式：

```lua
M.general = {
    n = {
        [";"] = { ":", "进入命令模式", opts = { nowait = true } },
    },
}
```

此映射为 NORMAL 状态 `n =` 输入字符 ++";"++ ，当在键盘上按下该字符时，会呈现字符 ++colon++ 。此字符是用于进入 COMMAND 模式的字符。还设置了选项 `nowait = true` 以立即进入该模式。这样，在使用 US QWERTY 布局的键盘上，我们就不需要使用 ++shift++ 来进入 COMMAND 模式了。

!!! Tip

    对于欧洲键盘（如意大利语键盘）的用户，建议将字符 ++";"++ 替换为 ++","++。

#### :material-language-lua: highlights.lua

该文件用于自定义编辑器的样式。此处编写的设置用于更改字体样式（**粗体**，*斜体*）、元素的背景色、前景色等方面。

### :material-folder-cog-outline: Configs 文件夹

此文件夹包含 **custom/plugins.lua** 文件中用于更改处理语言服务器（*lspconfig*）、linter/格式化程序（*conform*）以及覆盖 **treesitter**、**mason** 和 **nvim-tree** 基本设置（*override*）的插件的默认设置的所有配置文件。

```text
configs/
├── conform.lua
├── lspconfig.lua
└── overrides.lua
```

#### :material-language-lua: lspconfig.lua

*lspconfig.lua* 文件设置编辑器可以使用的本地语言服务器（LSP）。这将为支持的文件启用高级功能，例如自动补全或 snippets（代码片段），以快速创建代码片段。要将我们的 *lsp* 添加到配置中，只需编辑 NvChad 开发者专门准备的表格（在 *lua* 中，下面花括号中的内容代表一个表格）：

```lua
local servers = { "html", "cssls", "tsserver", "clangd" }
```

如你所见，默认已设置了一些服务器。要添加一个新的，在表格末尾输入即可。可用的服务器可以在 [mason packages](https://github.com/williamboman/mason.nvim/blob/main/PACKAGES.md) 中找到，其配置可参考 [lsp server configurations](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md)。

例如，如果我们还想支持 `yaml` 语言，可以如下例所示添加：

```lua
local servers = { "html", "cssls", "tsserver", "clangd", "yamlls" }
```

然而，更改文件并不会安装相应的语言服务器。这必须通过 *Mason* 单独安装。支持 *yaml* 的语言服务器是 [yaml-language-server](https://github.com/redhat-developer/yaml-language-server)，我们需要使用命令 `:MasonInstall yaml-language-server` 来安装。此时，我们就可以控制 Rocky Linux 文档页面头部（*frontmatter*）中编写的代码了。

#### :material-language-lua: conform.lua

 此文件配置了一些面向控制和格式化已编写代码的功能。编辑此文件比上一步需要更多关于配置的研究。可用组件的概览可以在 [buildins 页面](https://github.com/stevearc/conform.nvim/tree/master?tab=readme-ov-file#formatters)找到。

同样，创建了一个表格，即 ==formatters_by_ft== 表格，在其中输入自定义设置：

```lua
--type conform.options
local options = {
  lsp_fallback = true,

  formatters_by_ft = {
    lua = { "stylua" },

    javascript = { "prettier" },
    css = { "prettier" },
    html = { "prettier" },
    sh = { "shfmt" },
  },
}
```

如你所见，初始配置中只包含了标准格式化程序。例如，你可能需要 Markdown 语言的格式化程序，在这种情况下可以添加 [Markdownlint](https://github.com/DavidAnson/markdownlint)：

```lua
    markdown = { "markdownlint" },
```

同样，配置需要安装相应的软件包，通过 *Mason* 完成：

```text
:MasonInstall markdownlint
```

!!! Note

    此格式化程序的配置还需要在你的 home 文件夹中创建一个配置文件，本文档不再赘述。

#### :material-language-lua: overrides.lua

*overrides.lua* 文件包含对默认插件设置的更改。要应用更改的插件在 `custom/plugins.lua` 文件的 ==-- Override plugin definition options== 部分中通过使用 **opts** 选项（例如 `opts = overrides.mason`）来指定。

初始配置中有三个被标记为需要覆盖的插件，分别是 *treesitter*、*mason* 和 *nvim-tree*。暂时搁置 *nvim-tree*，我们将重点放在前两个上，它们允许我们显著改变编辑体验。

*treesitter* 是一个代码解析器，以交互方式负责代码格式化。每当我们保存一个被 *treesitter* 识别的文件时，它会被传递给解析器，解析器返回一个最佳缩进和高亮的代码树，使代码在编辑器中更易于阅读、理解和编辑。

处理此部分的代码如下：

```lua
M.treesitter = {
    ensure_installed = {
        "vim",
        "lua",
        "html",
        "css",
        "javascript",
        "typescript",
        "tsx",
        "c",
        "markdown",
        "markdown_inline",
    },
    indent = {
        enable = true,
        -- disable = {
        --   "python"
        -- },
    },
}
```

现在按照之前的示例，如果我们希望 Rocky Linux 文档页面的 *frontmatter* 能够正确高亮，可以在 `ensure_installed` 表格中最后一个解析器设置之后添加对 *yaml* 的支持：

```text
    ...
    "tsx",
    "c",
    "markdown",
    "markdown_inline",
    "yaml",
    ...
```

下次打开 NvChad 时，我们刚刚添加的解析器也会自动安装。

要在正在运行的 NvChad 实例中直接使用该解析器，我们随时可以通过以下命令安装，即使没有编辑文件也可以：

```text
:TSInstall yaml
```

文件中接下来是关于 *Mason* 安装服务器的部分。此表格中设置的所有服务器都可以通过 `:MasonInstallAll` 命令一次性安装（此命令在创建 *custom* 文件夹时也会被调用）。该部分如下：

```lua
M.mason = {
    ensure_installed = {
        -- lua stuff
        "lua-language-server",
        "stylua",

        -- web dev stuff
        "css-lsp",
        "html-lsp",
        "typescript-language-server",
        "deno",
        "prettier",
    },
}
```

同样，按照之前手动安装服务器以启用 *yaml* 支持的初始示例，我们可以通过将其添加到表格中来确保始终安装它：

```text
    ...
    "typescript-language-server",
    "deno",
    "prettier",

    -- yaml-language-server
    "yaml-language-server",
    ...
```

虽然在正在运行的 NvChad 实例中可以随时手动安装缺失的服务器，这一方面可能显得次要，但在将你的配置从一台机器迁移到另一台机器时却非常有用。

例如，假设我们已经配置好了 `custom` 文件夹，并希望将其转移到另一个 NvChad 安装中。如果我们已经配置了此文件，在复制或克隆我们的 `custom` 文件夹后，只需执行 `:MasonInstallAll` 就能在另一台机器上准备好所有服务器。


配置的最后部分，`M.nvimtree` 部分，负责配置 *nvim-tree*，启用显示与 git 仓库相关的文件树状态的功能：

```lua
  git = {
    enable = true,
  },
```

以及它们的高亮和相应图标：

```lua
  renderer = {
    highlight_git = true,
    icons = {
      show = {
        git = true,
      },
    },
  },
```

## :material-contain-end: 结论

NvChad 2.0 中引入的首次安装时可创建 `custom` 文件夹的功能，对所有首次接触此编辑器的用户来说无疑是一个巨大的帮助。对于那些已经接触过 NvChad 的用户来说，也节省了大量时间。

由于其引入以及 *Mason* 的使用，集成你自己的功能变得直接而快速。只需少量更改，你就可以立即使用 IDE 来编写代码。
