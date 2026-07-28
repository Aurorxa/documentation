---
title: Markdown 预览
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.7, 9.2
tags:
  - nvchad
  - plugins
  - editor
---

# Markdown 预览

## 简介

Markdown 语言被广泛应用于技术文档编写的原因之一是其可转换性。代码可以转换为多种格式显示（HTML、PDF、纯文本等），从而使内容能够在众多场景中使用。

具体来说，为 Rocky Linux 编写的文档会使用一个 *python* 应用程序转换为 `HTML`。该应用程序将用 *markdown* 编写的文档转换为静态 HTML 页面。

在为 Rocky Linux 编写文档时，会遇到一个问题：如何验证文档转换为 `HTML` 代码后的显示是否正确。

为了在你的编辑器中集成此功能，本页将介绍两个可用于此目的的插件：[toppair/peek.nvim](https://github.com/toppair/peek.nvim) 和 [markdown-preview.nvim](https://github.com/iamcco/markdown-preview.nvim)。这两个插件都支持 *github-style*、可以选择用于预览的浏览器，并且支持与编辑器同步滚动。

### Peek.nvim

[Peek](https://github.com/toppair/peek.nvim) 使用 [Deno](https://docs.deno.com/runtime/manual/)（一个 JavaScript、TypeScript 和 WebAssembly 运行时，具有默认的安全设置）来运行。默认情况下，除非显式启用，Deno 不允许任何文件、网络或环境访问。

要在编辑器配置中安装语言服务器，需要使用 *mason.nvim* 插件，该插件提供了 `:MasonInstall` 命令，可以自动引入和配置 *Deno*。

```text
:MasonInstall deno
```

!!! Warning

    在继续安装插件之前，**必须**先安装语言服务器。否则安装将失败，需要从 **plugins/init.lua** 中删除相关代码，通过打开 `Lazy` 并输入 ++"X"++ 来删除插件从而执行配置清理，然后重复安装过程。

要安装该插件，你需要编辑 **plugins/init.lua** 文件，添加以下代码块：

```lua
{
    "toppair/peek.nvim",
    build = "deno task --quiet build:fast",
    keys = {
        {
        "<leader>op",
            function()
            local peek = require("peek")
                if peek.is_open() then
            peek.close()
            else
            peek.open()
            end
        end,
        desc = "Peek (Markdown 预览)",
        },
    },
    opts = { theme = "dark", app = "browser" },
},
```

保存文件后，你可以通过 `:Lazy` 命令打开插件管理器界面来执行安装。插件管理器将已自动识别该插件，输入 ++"I"++ 即可安装。

但是，要获得完整功能，你必须关闭 NvChad（*nvim*）并重新打开。这是为了让编辑器将 **Peek** 的配置加载到编辑器中。

其配置中已包含激活命令 `<leader>op`，在键盘上对应 ++space++ + ++"o"++ 然后按 ++"p"++。

![Peek](./images/peek_command.png)

还有以下字符串：

```lua
opts = { theme = "dark", app = "browser" },
```

允许你传递预览的明暗主题选项以及用于显示的方法。

在此配置中，选择了 "browser" 方法，该方法会在系统默认浏览器中打开要查看的文件。但该插件还支持通过 "webview" 方法，仅使用 **Deno** 通过 [webview_deno](https://github.com/webview/webview_deno) 组件来预览文件。

![Peek Webview](./images/peek_webview.png)

### Markdown-preview.nvim

[Markdown-preview.nvim](https://github.com/iamcco/markdown-preview.nvim) 是一个用 `node.js`（JavaScript）编写的插件。它在 NvChad 上的安装不需要任何依赖项，因为开发者提供了在编辑器中完美运行的预编译版本。

要安装此版本，你需要将此代码块添加到 **plugins/init.lua**：

```lua
{
    "iamcco/markdown-preview.nvim",
    cmd = {"MarkdownPreview", "MarkdownPreviewStop"},
    lazy = false,
    build = function() vim.fn["mkdp#util#install"]() end,
    init = function()
        vim.g.mkdp_theme = 'dark'
    end
},
```

与前面的插件一样，你需要关闭编辑器并重新打开，以便 NvChad 加载新配置。同样，你可以向插件传递一些自定义选项，这些选项在项目仓库的[专用章节](https://github.com/iamcco/markdown-preview.nvim#markdownpreview-config)中有描述。

但是，这些选项必须修改为适配 `lazy.nvim` 的配置，尤其是此示例中配置的选项：

```lua
vim.g.mkdp_theme = "dark"
```

它对应于项目网站上描述的选项：

```lua
let g:mkdp_theme = 'dark'
```

如你所见，要设置选项，你必须修改它们的初始部分以使其可被解释。再举一个例子，我们来看看用于选择预览浏览器的选项，其指定方式如下：

```lua
let g:mkdp_browser = '/usr/bin/chromium-browser'
```

要在 NvChad 中正确解释此选项，需要将 `let g:` 替换为 `vim.g.` 来进行修改。

```lua
vim.g.mkdp_browser = "/usr/bin/chromium-browser"
```

这样，下次打开 NvChad 时，无论系统默认浏览器是什么，都将使用 `chromium-browser`。

该配置还提供了 `:MarkdownPreview` 和 `:MarkdownPreviewStop` 命令，分别用于打开和关闭预览。为了更快地访问这些命令，你可以将它们映射到 **mapping.lua** 文件，如下所示：

```lua
-- Markdown 预览映射
map("n", "<leader>mp", "<CMD> MarkdownPreview<CR>", { desc = "打开预览" })
map("n", "<leader>mc", "<CMD> MarkdownPreviewStop<CR>", { desc = "关闭预览" })
```

这将允许你通过输入 ++enter++ + ++"m"++ 然后 ++"p"++ 来打开 markdown 预览，并通过组合键 ++enter++ + ++"m"++ 然后 ++"c"++ 来关闭它。

!!! Note

    该插件还提供了 `:MarkdownPreviewToggle` 命令，但在撰写本文时，该命令似乎无法正常工作。如果你尝试调用它，它不会更改预览主题，而是会打开一个新的浏览器标签页，显示相同的预览。

![Markdown Preview](./images/markdown_preview_nvim.png)

## 结论与最终思考

对你正在编写的内容进行预览非常有用，无论是对新手还是对 Markdown 语言有更深入了解的用户都是如此。预览可以让你评估代码转换后的效果，以及其中包含的任何错误。

选择哪个插件完全取决于个人偏好，我们鼓励你尝试两者，以评估哪一个最适合你。

使用这些插件之一可以让你贡献符合所用代码规范的 Rocky Linux 文档，从而减轻文档审阅者的工作负担。
