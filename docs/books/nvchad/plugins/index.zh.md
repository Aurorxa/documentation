---
title: 概述
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.1
tags:
  - nvchad
  - plugins
  - editor
---

# 概述

## :material-message-outline: 简介

NvChad 开发者创建的自定义配置允许你拥有一个具有图形 IDE 许多功能的集成环境。这些功能通过插件构建到 Neovim 配置中。开发者选择用于 NvChad 的那些插件具有为通用用途设置编辑器的功能。

然而，Neovim 的插件生态系统要广阔得多，通过使用它们，可以扩展编辑器以专注于你自己的需求。

本节讨论的场景是创建 Rocky Linux 的文档，因此将解释用于编写 Markdown 代码、管理 Git 仓库以及与目的相关的其他任务的插件。

### :material-arrow-bottom-right-bold-outline: 要求

- 系统上正确安装了 NvChad
- 熟悉命令行
- 互联网连接可用

### :material-comment-processing-outline: 关于插件的一般提示

NvChad 的配置涉及从 `lua/plugins` 文件夹插入用户插件。其中初始是 **init.lua** 文件，其中安装了 *conform.nvim* 插件以及一些用于自定义系统功能的示例。  
虽然你可以将自己的插件放在文件中，但建议为自定义配置使用单独的文件。这样，你可以将初始文件用于任何对基本插件的覆盖，同时可以根据自己的偏好将插件组织到独立的文件中。

### :material-location-enter: 插入插件

配置查询 `plugins` 文件夹，其中的所有 *.lua* 文件都会被加载。这允许在从编辑器加载时合并多个配置文件。  
要正确插入，额外的文件必须将插件的配置包含在 ==lua tables== 中：

```lua title="lua table example"
return {
    { -- lua table
    -- your plugin here
    }, -- end lua table
}
```

还提供了一个 `configs` 文件夹，可以输入一些插件的特别长的设置或用户可修改的部分，例如 *conform.nvim* 的情况。

转向一个实际例子，假设我们想要将 [karb94/neoscroll.nvim](https://github.com/karb94/neoscroll.nvim) 插件包含到编辑器功能中，该插件允许在非常长的文件中改进滚动。  
对于其创建，我们可以选择创建一个 `plugins/editor.lua` 文件，将所有与编辑器使用相关的插件放在其中，或创建一个 `plugins/neoscroll.lua` 文件，将所有额外插件分开。

在此示例中，我们将遵循第一个选项，所以在 `plugins` 文件夹中创建一个文件：

```bash
touch ~/.config/nvim/lua/plugins/editor.lua
```

按照项目页面上的信息，我们将以下代码块插入其中：

```lua title="editor.lua"
return {
{
    "karb94/neoscroll.nvim",
    keys = { "<C-d>", "<C-u>" },
    opts = { mappings = {
        "<C-u>",
        "<C-d>",
    } },
},
}
```

一旦保存，它将被 NvChad 配置识别，并由 *lazy.nvim* 处理器使用其提供的功能负责其插入。

Neovim，作为 NvChad 配置的基础，没有集成自动配置更新机制与正在运行的编辑器。这意味着每次修改 plugins 文件时，都需要停止 `nvim` 然后重新打开它以获取插件的完整功能。

![plugins.lua](./images/plugins_lua.png)

## 总结与最终想法

Neovim 有一个庞大的插件生态系统，可以集成到 NvChad 中。对于研究来说，可以使用 [Dotfyle](https://dotfyle.com/) 网站，它提供关于 Neovim 插件和配置的信息，或者 [Neovimcraft](https://neovimcraft.com/)，它只关注可用插件。两者都提供关于插件的优秀通用信息以及指向 GitHub 上相应项目的链接。

从 2.5 版本开始引入的新插件搜索功能使得以非常高效且高度可配置的方式组织用户插件成为可能。在复杂配置中，它允许单独管理需要特殊配置（lua 代码或 autocmds）的插件，极大地简化了它们的管理。
