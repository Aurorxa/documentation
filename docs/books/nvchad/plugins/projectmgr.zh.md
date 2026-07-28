---
title: 项目管理器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.1
tags:
  - nvchad
  - plugins
  - editor
---

# 项目管理器

## 简介

IDE（集成开发环境）必须具备的功能之一就是能够管理开发者或编辑者所处理的各种项目。能够在打开 NvChad 后选择要处理的项目，而无需在 *statusline* 中输入命令来达到目的。这可以节省时间，并在项目数量较多时简化管理。

使用 [charludo/projectmgr.nvim](https://github.com/charludo/projectmgr.nvim) 将集成此功能。该插件提供了与 `Telescope` 的优秀集成以及一些有趣的附加功能，例如在打开 *项目* 时能够同步 *git* 仓库。

该插件还会跟踪编辑器关闭时的状态，使你下次打开时能够恢复之前正在处理的所有页面。

### 插件安装

要安装该插件，你需要编辑 **plugins/init.lua** 文件，添加以下代码块：

```lua
{
    "charludo/projectmgr.nvim",
    lazy = false, -- 重要！
},
```

保存文件后，该插件将可供安装。要安装它，请使用 `:Lazy` 命令打开 *lazy.nvim* 并输入 ++"I"++。安装完成后，你需要退出编辑器并重新打开，以便让它读取你输入的新配置。

该插件提供单一命令 `:ProjectMgr`，该命令打开一个交互式缓冲区，你可以通过键盘快捷键执行所有操作。首次打开时，缓冲区将为空，如下截图所示：

![ProjectMgr Init](./images/projectmgr_init.png)

### 使用项目管理器

所有操作都通过 ++ctrl++ 键加一个字母（例如 `<C-a`）来执行，而 `<CR>` 键对应 ++enter++ 键。

下表显示了所有可用的操作：

| 键       | 操作                                             |
|:--------:| ----------------------------------------------- |
| `<CR>`   | 打开光标下的项目                                  |
| `<C-a>`  | 通过交互式过程添加项目                             |
| `<C-d>`  | 删除项目                                          |
| `<C-e>`  | 更改项目设置                                       |
| `<C-q>`  | 关闭缓冲区                                        |

要添加你的第一个项目，你需要使用组合键 ++ctrl++ + ++"a"++，这将在 *statusline* 中打开一个交互式菜单。在此示例中，我们将使用保存在 **~/lab/rockydocs/documentation** 中的 Rocky Linux 文档克隆。

第一个问题会询问你项目的名称：

> 项目名称：documentation

接着会询问项目路径：

> 项目路径：~/lab/rockydocs/documentation/

接下来可以设置在打开和关闭项目时要运行的命令。这些命令指的是在编辑器中可执行的命令，而非 **bash** 语言。

例如，你可以在打开编辑器时使用 `NvimTreeToggle` 命令同时打开一个包含 *NvimTree* 的侧缓冲区。

> 启动命令（可选）：NvimTreeToggle

或者运行一个在关闭编辑器之前执行的命令。

> 退出命令（可选）：

命令输入时应省略在 *statusline* 中运行相同命令时使用的冒号 `:`。

配置完成后，你的项目将出现在缓冲区中。要打开它，选择该项目并按 ++enter++。

![ProjectMgr Add](./images/projectmgr_add.png)

从截图中可以看到，在 **Config & Info** 部分，该插件已识别出该文件夹由 *Git* 管理，并提供了一些相关信息。

编辑项目通过 ++ctrl++ + ++"e"++ 完成，包含一个新的交互循环。而任何删除操作通过 ++ctrl++ + ++"d"++ 组合键完成。

### 附加功能

该插件提供了一些在[专用章节](https://github.com/charludo/projectmgr.nvim#%EF%B8%8F-configuration)中指定的附加功能。最有趣的包括在打开项目时同步 git 仓库的能力，以及在关闭编辑器时存储编辑器状态的能力。这两项功能已经存在于默认配置文件中，尽管关于 *Git* 的功能是禁用的。

要在打开项目时添加仓库同步，你需要将以下代码添加到初始插件配置中：

```lua
config = function()
    require("projectmgr").setup({
        autogit = {
            enabled = true,
            command = "git pull --ff-only >> .git/fastforward.log 2>&1",
        },
    })
end,
```

从代码中可以看出，调用了 `require("projectmgr").setup` 函数，它允许你覆盖默认设置。你在其中设置的所有内容都会改变其行为方式。

`git pull --ff-only` 命令执行仓库的 *fast forward* 同步，仅下载那些没有冲突且无需你干预即可更新的文件。

该命令的结果还被重定向到文件 **.git/fastforward.log**，以防止其显示在运行 NvChad 的终端上，并且可以保留同步历史记录。

还提供了保存关闭时会话的选项。这使你能够在选择项目并重新打开时返回到之前正在处理的页面。

```lua
session = { enabled = true, file = "Session.vim" },
```

此选项默认启用，但它将 **Session.vim** 文件写入项目 *根* 目录，这在 Rocky Linux 文档的情况下并不理想。在此示例中，它被保存在不受版本控制的 `.git` 文件夹中。

根据你的需要调整 **Session.vim** 和 **fastforward.log** 的路径。

完成更改后，你的配置应如下所示：

```lua
{
    "charludo/projectmgr.nvim",
    lazy = false, -- 重要！
    config = function()
        require("projectmgr").setup({
            autogit = {
                enabled = true,
                command = "git pull --ff-only > .git/fastforward.log 2>&1",
            },
            session = { enabled = true, file = ".git/Session.vim" },
        })
    end,
},
```

现在，每次你打开项目时，更新后的文件都会从 Git 仓库自动下载，并且你之前在编辑器中处理的文件将打开，随时可以编辑。

!!! Warning

    NvChad 保存的会话缓冲区中打开的文件不会自动更新。

要检查打开的文件是否与从仓库更新的文件不匹配，你可以使用 `:checktime` 命令，它检查编辑器中打开的文件是否已在 NvChad 外部修改，并提醒你需要更新缓冲区。

### 快捷键映射

为了加快打开项目的速度，你可以创建一个键盘快捷键并将其放入 **mapping.lua** 中的映射中。示例如下：

```lua
-- 项目
map("n", "<leader>fp", "<CMD> ProjectMgr<CR>", { desc = "打开项目" })
```

在编辑器处于 **NORMAL** 状态时，你可以通过组合键 ++space++ + ++"f"++ 然后 ++"p"++ 来打开项目管理器。

## 结论与最终思考

随着你正在处理的项目数量增多，拥有一个工具来帮助你管理它们可能会非常有用。此插件将帮助你加快工作速度，减少访问需要编辑的文件所需的时间。

还应指出的是，它与 `Telescope` 的出色集成使项目管理非常高效。
