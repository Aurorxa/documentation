---
title: Marksman
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested with: 8.8, 9.2
tags:
  - nvchad
  - editor
  - markdown
---

# Marksman - 代码助手

Marksman 是在为 Rocky Linux 起草文档时的有用工具。它允许轻松输入定义 *markdown* 语言标签所需的符号。这使你能够更快地编写并减少出错的可能性。

NvChad/Neovim 已经包含了帮助写作的文本小部件，例如按输入频率索引的常用词重复。此语言服务器包含的新选项将丰富这些小部件。

[Marksman](https://github.com/artempyanykh/marksman) 与你的编辑器集成，通过 [LSP 协议](https://microsoft.github.io/language-server-protocol/)帮助你编写和维护 Markdown 文档，从而提供诸如自动完成、转到定义、引用搜索、名称重构、诊断等功能。

## 目标

- 提高 NvChad 编写 Markdown 代码的效率
- 生成符合 Markdown 语言规则的文档
- 加深你对语言本身的认识

## 要求和技能

- 对 Markdown 语言有基本了解，建议阅读 [Markdown 指南](https://www.markdownguide.org/)
- 在使用的机器上正确安装了 NvChad

**难度等级** :star:

**阅读时间：** 20 分钟

## Marksman 的安装

安装此语言服务器不会遇到任何特别的问题，因为它在 **Mason** 中天然可用。通过以下命令直接从 *statusline* 安装：

`:MasonInstall marksman`

该命令将打开 *Mason* 界面并直接安装所需的语言服务器。一旦二进制文件安装完成，你可以用 ++"q"++ 键关闭 *Mason* 屏幕。

然而，它的安装还未包含将其集成到编辑器中。要启用它，必须将其放入配置的 `configs/lspconfig.lua` 文件中。

## 集成到编辑器中

!!! note "NvChad 中的 LSP"

    [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) 插件将语言服务器集成到 NvChad 中。此插件极大地简化了它们在 NvChad 配置中的包含。

*lspconfig.lua* 文件负责输入使用语言服务器所需的调用，并允许你指定已安装的服务器。要将 *marksman* 集成到编辑器的语言服务器配置中，你需要通过添加新的 LSP 来编辑 *local servers* 字符串。

使用以下命令在文件上打开你的 NvChad：

```bash
nvim ~/.config/nvim/lua/configs/lspconfig.lua
```

并编辑 *local servers* 字符串，完成后将如下所示：

```lua
local servers = { "html", "cssls", "tsserver", "clangd", "marksman" }
```

保存文件并使用 `:wq` 命令关闭编辑器。

要检查语言服务器是否正确激活，在你的 NvChad 中打开一个 markdown 文件并使用 `:LspInfo` 命令查看应用于该文件的语言服务器。在摘要中应该会看到类似以下内容：

```text
 Client: marksman (id: 2, bufnr: [11, 156])
   filetypes:       markdown
   autostart:       true
   root directory:  /home/your_user/your_path/your_directory
   cmd:             /home/your_user/.local/share/nvim/mason/bin/marksman server
 
 Configured servers list: cssls, tsserver, clangd, html, yamlls, lua_ls, marksman
```

这表明 *marksman* 服务器已为打开的文件激活，并且它自动启动（`autostart: true`），因为它将文件识别为 markdown 文件 `filetypes: markdown`。其他信息指示用于代码检查的可执行文件路径 `cmd:`，它使用 `marksman server` 模式，并使用根目录 `your_directory` 进行检查。

!!! note "根文件夹"

    "根文件夹"的概念在使用语言服务器时很重要，因为要对文档执行控制，例如链接到其他文件或图像，它必须具有项目的"全局视图"。我们可以说"*root folders*"等同于图形 IDE 中的"*Projects*"。

    编辑器为打开文件使用的*根目录*，也称为"*工作目录*"，可以通过 `:pwd` 命令查看。如果它与期望的不匹配，可以使用 `:lcd` 命令更改。此命令仅为该缓冲区重新分配*工作目录*，而不改变编辑器中打开的其他缓冲区的任何设置。

## Marksman 的使用

一旦你完成了输入它的所有步骤，语言服务器将在打开编辑器中的 *markdown* 文件时激活。进入 `INSERT` 模式，当输入某些字符时，你将在小部件中获得新选项，帮助你编写文档。在下面的截图中，你可以看到这些小部件中可用的一些 markdown 代码片段。

![Marksman Snippets](./images/marksman_snippets.png)

## 主要按键

语言服务器提供了许多激活写作辅助的快捷方式。这包括快速插入 Markdown 标签、创建链接以及将图像插入文档。下面是一份非详尽的字符列表，这些字符激活了各种代码片段。

这些代码片段显示在也包含其他快捷方式的小部件中。使用 ++tab++ 键在小部件中导航，选择 *marksman* 提供的那些。

| 按键 | 代码片段 |
|--------------- | --------------- |
| ++"h"++ | 允许快速输入标题标题（*h1* 到 *h6*），例如，输入 *h4* 并按回车将插入四个井号和空格，光标已经就位以输入你的标题 |
| ++"b"++ | 输入此字符激活使用快捷方式输入粗体文本的能力，通过插入四个星号并将光标放在中间，使得编写 **粗体** 部分更快 |
| ++"i"++ | 与前面的字符一样，它允许你选择快速插入 *斜体* 文本，通过输入两个星号并将光标放在中间。 |
| ++"bi"++ | 此按键插入六个星号，将光标放在中间，用于编写 ***粗斜体*** 文本 |
| ++"img"++ | 此按键插入用于在文档中插入图像的 markdown 结构，格式为 `![alt text](path)`。请注意，编写路径可以使用服务器提供的自动完成功能。 |
| ++"link"++ | 此按键创建 `[text](url)` 链接的 markdown 标签结构。同样，如果链接引用的是 **工作目录** 中的文件，你将能够使用自动完成功能，服务器将检查引用的正确性。 |
| ++"list"++ | 输入此按键允许输入三项列表以开始创建编号或无序列列表 |
| ++"q"++ | 此字符允许插入引用标签 `>`，后跟空格，并将光标定位在引用的书写位置 |
| ++"s"++ | 此字符激活多种可能性，包括插入四个波浪号并将光标放在中间，用于编写 ~~删除线~~ 文本 |
| ++"sup"++ | 此按键插入 *上标* 标签。商标<sup>TM |
| ++"sub"++ | 此按键插入 *下标* 标签。注释<sub>1 |
| ++"table"++ | 此按键启用快速创建表格结构，并允许你从许多起始结构中选择 |
| ++"code"++ | 通过将光标放在两个反引号所在位置的中心，插入内联代码块 |
| ++"codeblock"++ | 插入三行，两行带有三个反引号，一行为空，你可以在其中插入代码块。请注意，它还插入了字符串 *language*，需用你在块中使用的语言填写。 |

!!! note "代码块声明"

    Markdown 代码规则建议始终声明块中使用的代码，即使没有高亮功能也要声明，以便正确解释。如果其中的代码过于泛化，建议使用 "text" 进行声明。

Markdown 标签快捷方式的激活按键还包括其他组合，你可以在使用语言服务器的过程中发现它们。

## 总结

虽然不是严格必需的，但随着时间的推移，这个语言服务器可以成为你为 Rocky Linux 编写文档的好伙伴。

通过使用它并随后记住插入 Markdown 代码符号的主要按键，它将实现更有效的快速写作，让你将注意力集中在内容上。
