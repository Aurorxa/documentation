---
title: 插件管理器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.1
tags:
  - nvchad
  - coding
  - plugins
---

# :material-file-settings-outline: 插件管理器

NvChad 2.0 中的插件管理由 [folke/lazy.nvim](https://github.com/folke/lazy.nvim) 完成，此插件在首次安装时通过引导过程安装。该插件允许你对插件执行所有常见操作，如安装、更新等。

![Lazy Nvim](../images/lazy_nvim.png)

## :material-application-import: 主要功能

- 从统一界面管理插件上的所有操作。
- 通过缓存和编译 Lua 模块字节码优化插件性能。
- 启动时自动检查并安装缺失的插件，这在将配置从一台机器转移到另一台机器时非常有用。
- 用于查看插件加载时间的分析器。允许你监控和排查由故障插件引起的问题。
- 通过在 *lazy-lock.json* 文件中存储所有已安装插件的修订版，实现多工作站间的插件同步。

## :material-arrow-bottom-right-bold-outline: 预备操作

*lazy.nvim* 集成了环境健康检查功能，可以通过命令 `:checkhealth lazy` 调用。该命令应在新的缓冲区中返回类似以下内容：

```text
lazy: require("lazy.health").check()
========================================================================
## lazy.nvim
  - OK: Git installed
  - OK: no existing packages found by other package managers
  - OK: packer_compiled.lua not found
  - WARNING: {nvim-lspconfig}: overriding <config>
```

虽然不是严格必需的，但在开始处理我们的自定义配置之前检查构建环境允许我们从任何可能发生在插件本身或编写其配置中的错误或故障中排除此变量。

查看插件本身提供的内联帮助也可能很有用。要打开它，我们可以使用 `:Lazy help` 命令，或通过输入 ++"?"++ 从插件界面调用。

![Lazy Help](../images/lazy_help.png)

帮助提供了界面导航、控件及其功能的介绍。

现在，在检查了环境并获得了基本知识后，我们可以继续创建自己的配置。目的显然是向编辑器添加功能以满足我们的需求，并且由于这是通过将插件包含在 NvChad 配置中来实现的，我们将从添加一个插件开始。

## :material-location-enter: 插入插件

虽然可以通过 *lazy.nvim* 界面方便地执行已安装插件的管理，但插入新插件需要手动编辑 **lua/plugins/init.lua** 文件。

在此示例中，我们将安装 [natecraddock/workspaces.nvim](https://github.com/natecraddock/workspaces.nvim) 插件。此插件允许你保存并在以后使用工作会话（workspaces），以便快速访问它们。我们使用以下命令打开文件：

```bash
nvim ~/.config/nvim/lua/plugins/init.lua
```

并在 *better-escape.nvim* 插件之后插入以下代码：

```lua
    -- Workspaces
    {
        "natecraddock/workspaces.nvim",
        cmd = { "WorkspacesList", "WorkspacesAdd", "WorkspacesOpen", "WorkspacesRemove" },
        config = function()
            require("workspaces").setup {
        hooks = {
            open = "Telescope find_files",
        },
      }
    end,
    },
```

一旦文件保存，我们将收到一条通知，要求我们批准：

```text
# Config Change Detected. Reloading...

- **changed**: `lua/plugins/init.lua`
```

这是通过内置于 *lazy.nvim* 中的机制实现的，该机制检查插件及其配置的状态，从而无需退出编辑器就可以对插件执行操作。

显然，我们将回答 "yes"。

现在，如果我们使用 `:Lazy` 命令打开插件管理器，我们将发现我们的插件已被识别，并准备好安装。要安装它，只需输入 ++"I"++

![Install Plugin](../images/lazy_install.png)

此时 *lazy.nvim* 将负责将仓库下载到路径 **.local/share/nvim/lazy/** 并执行构建。安装完成后，我们将拥有一个名为 *workspaces.nvim* 的新文件夹：

```text
.local/share/nvim/lazy/workspaces.nvim/
├── CHANGELOG.md
├── doc
│   ├── tags
│   └── workspaces.txt
├── LICENSE
├── lua
│   ├── telescope
│   │   └── _extensions
│   │       └── workspaces.lua
│   └── workspaces
│       ├── init.lua
│       └── util.lua
├── README.md
└── stylua.toml
```

我们现在将拥有可以通过数组中设置的命令调用的插件功能：

```lua
cmd = { "WorkspacesList", "WorkspacesAdd", "WorkspacesOpen", "WorkspacesRemove" },
```

输入还涉及在 *lazy-lock.json* 文件中添加一个字符串，用于状态跟踪和后续更新。*lazy-lock.json* 文件的功能将在下面的相应部分中描述。

```json
  "workspaces.nvim": { "branch": "master", "commit": "dd9574c8a6fbd4910bf298fcd1175a0222e9a09d" },
```

## :material-tray-remove: 删除插件

与安装一样，从配置中删除插件需要通过手动编辑 *lua/plugins/init.lua* 文件。为了遵循示例，我们将删除刚才安装的插件。

我们打开编辑器并从配置中删除插件。这可以方便地通过用鼠标选择要删除的行，然后按 ++"x"++ 删除它们，再按 ++ctrl++ + ++"s"++ 保存文件来完成。

![Remove Plugin](../images/remove_plugin_01.png)

我们再次会收到关于 *init.lua* 文件修改的通知，回答 "yes"，一旦我们打开 *Lazy*，我们的插件将被标记为要删除。删除通过按 ++"X"++ 键完成。

![Lazy Clean](../images/remove_plugin_02.png)

删除一个插件基本上包括删除安装期间创建的文件夹。

## 更新插件

一旦插件被安装和配置，它们由 *lazy.nvim* 独立管理。要检查更新，只需打开管理器并输入 ++"C"++。*Lazy* 将检查已安装插件的仓库（*git fetch*），然后呈现一个可更新的插件列表，检查后，可以使用 ++"U"++ 一次性全部更新，或使用 ++"u"++ 选择后单独更新。

![Lazy Check](../images/lazy_check.png)

!!! note

    即使上面的截图没有显示，如果有包含 "breaking changes" 提交的插件，它们将首先显示。

还可以仅使用 `Sync` 命令运行整个更新周期。从界面中，输入 ++"S"++ 或使用命令 `:Lazy sync`，我们将调用该函数，它由 `install` + `clean` + `update` 的串联组成。

更新过程，无论是单独完成还是累计完成，也会修改 *lazy-lock.json* 文件。特别是，提交将被修改以与 GitHub 上仓库的状态同步。

## 附加功能

在编写插件时，特别关注了性能和代码效率，并为我们提供了一种评估各种插件启动时间的方法。我们获得了一个 *profiler*（分析器），可以通过 `:Lazy profile` 命令或从界面使用 ++"P"++ 键调用。

![Lazy Profiler](../images/lazy_profile.png)

在这里，我们可以看到各种插件的加载时间，可以按组合键 ++ctrl++ + ++"s"++ 按配置中的条目或加载时间排序。我们还可以通过使用组合键 ++ctrl++ + ++"f"++ 设置最小阈值（毫秒）来搜索插件的加载时间。

这些信息在编辑器异常变慢时对于故障排除可能很有用。

该插件还提供了对插件上最后执行的操作的显示，可以从界面使用 ++"L"++ 键或从编辑器本身使用 `:Lazy log` 命令调用此显示。

![Lazy Log](../images/lazy_log.png)

它还集成了调试功能，允许我们检查活动的延迟加载处理程序以及模块缓存中的内容。要激活它，我们可以从界面使用 ++"D"++ 键，或使用 `:Lazy debug` 命令调用它。

![Lazy Debug](../images/lazy_debug.png)

## 同步

Lazy.nvim 允许通过在 *json* 文件中存储所有已安装插件的状态来同步它们。在其中，为每个插件创建一个字符串，该字符串包含在 **~/.local/share/nvim/lazy/** 中找到的对应已安装插件的文件夹名称、相应的分支以及用于从 GitHub 仓库同步的提交。用于此目的的文件是位于 **~/.config/nvim** 根文件夹中的 `lazy-lock.json` 文件。下面是文件的一个摘录：

```json
{
  "Comment.nvim": { "branch": "master", "commit": "8d3aa5c22c2d45e788c7a5fe13ad77368b783c20" },
  "LuaSnip": { "branch": "master", "commit": "025886915e7a1442019f467e0ae2847a7cf6bf1a" },
  "base46": { "branch": "v2.0", "commit": "eea1c3155a188953008bbff031893aa8cb0610e9" },
  "better-escape.nvim": { "branch": "master", "commit": "426d29708064d5b1bfbb040424651c92af1f3f64" },
  "cmp-buffer": { "branch": "main", "commit": "3022dbc9166796b644a841a02de8dd1cc1d311fa" },
  "cmp-nvim-lsp": { "branch": "main", "commit": "0e6b2ed705ddcff9738ec4ea838141654f12eeef" },
  "cmp-nvim-lua": { "branch": "main", "commit": "f3491638d123cfd2c8048aefaf66d246ff250ca6" },
  "cmp-path": { "branch": "main", "commit": "91ff86cd9c29299a64f968ebb45846c485725f23" },
  "cmp_luasnip": { "branch": "master", "commit": "18095520391186d634a0045dacaa346291096566" },
...
```

由于提交的存储，我们可以准确地看到插件在安装或更新时的仓库状态。这允许我们通过 `restore` 函数将其带回或将其带到编辑器中的相同状态。该函数可以从界面使用 ++"R"++ 键或使用 `:Lazy restore` 调用，将编辑器中的所有插件更新到 *lazy-lock.json* 文件中定义的状态。

通过将 *lazy-lock.json* 文件从稳定配置复制到一个安全的地方，如果更新产生了问题，我们有能力将编辑器恢复到该状态。通过将其导出到另一个工作站，我们可以使用它来配置具有相同功能的编辑器。

如果我们将其置于版本控制之下，我们可以在我们使用的所有工作站之间实现配置同步。

现在，在说明了插件管理器之后，我们可以继续分析用户界面。
