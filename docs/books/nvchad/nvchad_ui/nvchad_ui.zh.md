---
title: NvChad 用户界面
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested: 8.6, 9.0
tags:
  - nvchad
  - coding
  - nvchad interface
---

# NvChad 界面

!!! note "关于 `mappings.lua` 的几句话"

    在 NvChad 的官方文档中，提到了诸如 `<leader>` 等命令，以及其他可能对普通读者不太易懂的按键。原因是默认的 `mappings.lua` 文件（`../nvim/lua/core/mappings.lua`）。

    此文件及其描述的键映射在[此处](https://nvchad.com/docs/config/mappings)进行了概述。需要明确的是，你可以使用自定义的 `mappings.lua` 文件（`../nvim/lua/custom/mappings.lua`）覆盖任何键映射。

    为避免你现在正在阅读的文档中出现混淆，我们假设你使用的是默认的 `mappings.lua` 文件，并将替换你需要输入以访问功能的实际命令。标准按键引用如下：

    * leader = ++space++
    * A = ++alt++
    * C = ++ctrl++
    * S = ++shift++

    如果某个命令指定了 `<leader>uu` 作为例子，我们将替换为实际的按键组合 ++space++++"uu"++。

    这将在下一节[使用 NvChad](./using_nvchad.md)中再次涉及。

一旦安装了 Neovim 并输入了 NvChad 配置，我们的 IDE 应该如下所示：

![NvChad Default](../images/ui_default.png)

该界面已经具有一些高级功能，例如指示 git 仓库的状态，但可以通过使用语言服务器进一步增强，并通过覆盖一些基本配置进行自定义。构成它的基本模块将在下面详细说明。

## Tabufline

![Tabufline](../images/ui_tabufline.png)

用户界面在顶部呈现一个名为 `Tabufline` 的栏，其中管理着打开的缓冲区。`Tabufline` 仅在你打开多个文件时才显示。打开的缓冲区显示文件类型图标、文件名及其状态。状态通过图标表示。

如果如截图中所示，有一个红色的 `x`，表示该文件可以关闭因为它已经保存。如果图标是绿色的 `.`，则表示该文件需要保存，而关闭命令 ++shift++ + ++":"++ + ++"q"++ 将产生警告："No write since last change"。

右侧是设置 _dark_ 或 _light_ 主题的图标。通过用鼠标点击它，我们可以选择我们想要的主题。

![NvChad Light](../images/ui_default_light.png)

右侧还有关闭编辑器的图标。

## 中间部分 - 打开的缓冲区

编辑器的中心部分由此时编辑器上活动的缓冲区（_index.en.md_）组成。要引入一些额外的功能，我们可以通过在示例中再打开一个文件（_index.it.md_），在分割的缓冲区中同时处理两个文件。

在编辑器中，我们将有第一个缓冲区在前台，第二个缓冲区在 Tabufline 中列出。现在，如果我们使用 ++shift++ + ++":"++ + **vsplit** 命令分割第一个缓冲区，并选择右侧缓冲区，点击 tabufline 中第二个文件（_index.it.md_）的名称，它将在右侧缓冲区中打开，我们可以并排处理这两个文件。

![NvChad Split](../images/ui_nvchad_split.png)

## Statusline

![Statusline](../images/ui_statusline.png)

在底部我们看到 Statusline，它处理状态信息。右侧我们看到编辑器状态。我们必须记住，我们正在使用一个文本编辑器，特别是它保持了 Vim 的理念和操作方式。可能的状态有：

- **NORMAL**
- **INSERT**
- **COMMAND**
- **VISUAL**

编辑文档从 **NORMAL** 模式开始，在此模式下你打开文件，然后你可以切换到 **INSERT** 模式进行编辑，完成后按 ++esc++ 退出并返回 **NORMAL** 模式。

现在要保存文件，你通过输入 `:` 后跟 `w`（_write_）切换到 **COMMAND** 模式以写入它，然后按 ++esc++ 返回 **NORMAL** 模式。状态指示在学习如何使用它时非常有用，尤其是如果一个人不太熟悉 Vim 工作流。

然后我们看到打开的文件名，如果我们正在处理一个 git 仓库，我们将有仓库状态的指示。这是通过 _lewis6991/gitsigns.nvim_ 插件实现的。

转向右侧，我们看到打开编辑器时所在的文件夹名称。在使用 LSP 的情况下，这表示被视为 `workspace` 的文件夹，因此在诊断期间以及跟踪光标在文件中的位置时会评估该文件夹。

## 集成帮助

NvChad 和 Neovim 提供了一些有用的命令，用于显示预设的按键组合和可用选项。

如果单独按下 ++space++ 键，它将提供关联命令的图例，如下面的截图所示：

![Space Key](../images/ui_escape_key.png)

要查看编辑器中包含的所有命令，我们可以使用 ++space++ + ++"wK"++ 命令，将产生以下结果：

![leader wK](../images/ui_wK_key.png)

按下 ++"d"++ 我们可以显示剩余的命令：

![leader wK d](../images/ui_wK_01.png)

如我们所见，几乎所有命令都涉及到文档或缓冲区内的导航。没有包含打开文件的命令。这些由 Neovim 提供。

要查看所有 Neovim 的选项，可以使用 ++shift++ + ++":"++ + **options** 命令，它将呈现一个按类别索引的选项树。

![Nvim Options](../images/nvim_options.png)

这为我们提供了一种通过内置帮助在使用编辑器时学习命令，并深入研究可用选项的方法。

## NvimTree

为了处理我们的文件，我们需要一个文件资源管理器，这由 _kyazdani42/nvim-tree.lua_ 插件提供。使用组合键 ++ctrl++ + ++"n"++ 我们可以打开 NvimTree。

![NvimTree](../images/nvim_tree.png)

NvimTree 的命令和功能的详细描述可以在[专属页面](nvimtree.md)上找到。

现在我们已经探索了界面组件，我们可以继续使用 NvChad 了。
