---
title: 在 NvChad 中使用 vale
author: Steven Spencer
contributors: Franco Colussi, Krista Burdine, Serge, Ganna Zhyrnova
tags:
  - vale
  - linters
---

# NvChad（Neovim）中的 `vale`

## :material-message-outline: 简介

`vale.sh` 是面向希望提高文风和一致性技术写作的领先开源项目之一。它可以在几乎所有主流操作系统平台（Linux、macOS、Windows）的众多编辑器中使用。你可以前往 [vale.sh 网站](https://vale.sh/)了解更多关于该项目的信息。本指南将带你完成将 `vale` 添加到 NvChad 的过程。由于它包含在用于安装的 Mason 软件包中，过程并不太难，但确实需要一些小的编辑和配置才能使其运行。需要明确的是，NvChad 实际上是编辑器 Neovim 的配置管理器，因此从现在开始，将使用 `nvim` 作为引用。

## :material-arrow-bottom-right-bold-outline: 前提条件

* 熟悉 NvChad 2.0 会有所帮助
* 能够使用你喜欢的编辑器（`vi` 或你喜欢的编辑器）从命令行更改文件
* NvChad 中已正确安装 *nvim-lint* 插件

### :material-monitor-arrow-down-variant: 安装 nvim-lint

[nvim-lint](https://github.com/mfussenegger/nvim-lint) 插件支持将 ==linters== 插入编辑器中，通过纠正代码或内容的语法和语义部分来辅助编写。

要安装 *nvim-lint* 插件，只需在 `lua/plugins` 文件夹中创建一个 **nvim-lint.lua** 文件；当 Neovim 实例下次启动时，该文件将被集成到配置中。

文件内容如下：

```lua title="nvim-lint.lua"
return {
 {
  "mfussenegger/nvim-lint",
  enabled = true,
  event = "VeryLazy",
  config = function()
   require("lint").linters_by_ft = {
    markdown = { "markdownlint", "vale" },
    yaml = { "yamllint" },
   }

   vim.api.nvim_create_autocmd({ "BufWritePost" }, {
    callback = function()
     require("lint").try_lint()
    end,
   })
  end,
 },
}
```

此配置文件设置为与 markdown 代码配合使用，但可以根据项目网站上[可用的 linters](https://github.com/mfussenegger/nvim-lint?tab=readme-ov-file#available-linters)进行修改或扩展。

完成更改后，退出并重新进入 NvChad 以安装插件并导入配置。

## :material-monitor-arrow-down-variant: 使用 Mason 安装 `vale`

在 NvChad 中使用 Mason 安装 `vale`，只需几个额外步骤就能使软件包保持最新。在 `nvim` 中定期运行 Mason 会显示是否有需要安装的更新，并允许你从那里更新它们。一旦安装，这也包括 `vale`。让我们从运行 `nvim` 开始，打开一个空文件，然后使用 ++shift++ + ++":"++ + Mason 进入命令模式，这应该会显示类似以下界面：

![vale_mason](images/vale_mason.png)

与其查看整个软件包列表，不如使用菜单项 4 将列表限定为 linters。按 ++4++ 并在列表中向下滚动，直到找到 `vale`，然后将光标放在该行上，按 ++"i"++ 安装。你的列表现在应该显示 `vale` 已安装：

![vale_mason_installed](images/vale_mason_installed.png)

### :material-timer-cog-outline: 配置和初始化 `vale`

你可以使用两种方法来配置 `vale`。你可以从以下两个选项中挑选你喜欢的方法。一种是在 `vale` 二进制文件的路径内创建配置文件，然后将它们移动到你的 home 文件夹；另一种是直接在你的 home 文件夹中创建配置文件。两种方法效果相同。第二种方法手动步骤较少，但需要指定 `vale` 二进制文件的完整路径。

!!! tip

    如果你想隐藏你的 "styles" 文件夹（见下文），在创建过程中只需轻微修改 `.vale.ini` 的内容，将 "StylesPath" 选项从 "styles" 改为隐藏名称，如 ".styles" 或 ".vale_styles"。示例如下：

    ```
    StylesPath = .vale_styles
    ```

仅安装 `vale` 还不够。你还需要一些额外的项目。首先，你需要一个位于 home 文件夹根目录的 `.vale.ini` 文件。其次，你需要使用 `vale sync` 生成 "styles" 目录。

=== "从 `vale` 二进制文件的路径内安装"

    如果你在 `vale` 二进制文件的路径内：`~/.local/share/nvim/mason/packages/vale/`，你可以在此处创建 `.vale.ini` 文件，生成 "styles" 目录，然后将它们都移动到你的 home 根目录 `~/`。使用 [`vale.sh` 网站](https://vale.sh/generator)上的配置工具可以轻松创建 `.vale.ini` 文件。在这里，选择 "Red Hat Documentation Style Guide" 作为基础风格，选择 "alex" 作为辅助风格。使用 'alex' 是可选的，但可以帮助你捕捉和修正性别化、极化或种族相关的词汇等，这很重要。如果你选择了这些选项，你的屏幕应如下所示：

    ![vale_ini_nvchad](images/vale_ini_nvchad.png)

    只需复制底部的内容，用你喜欢的编辑器创建 `.vale.ini` 文件，然后粘贴你复制的内容。

    你需要创建 "styles" 文件夹。通过使用 `sync` 命令运行 `vale` 二进制文件来完成此操作。同样，如果你是从 `~/.local/share/nvim/mason/packages/vale/` 目录执行此操作，只需运行：

    ```bash
    ./vale sync
    ```

    完成后将显示以下内容：

    ![vale_sync](images/vale_sync.png)

    将 `.vale.ini` 和 `styles` 文件夹复制到你的 home 文件夹根目录：

    ```bash
    cp .vale.ini ~/
    cp -rf styles ~/
    ```

=== "从你的 home 目录安装"

    如果你不想复制文件，而是想直接在 home 目录中创建它们，可以从 `~/` 使用以下命令：

    首先，使用 [`vale.sh` 网站](https://vale.sh/generator)在你的 home 文件夹中创建 `.vale.ini`。同样，选择 "Red Hat Documentation Style Guide" 作为基础风格，选择 "alex" 作为辅助风格。然后将内容复制到 `.vale.ini` 文件中。

    ![vale_ini_nvchad](images/vale_ini_nvchad.png)

    接下来，运行 `vale sync` 命令。由于你在 home 目录中，需要使用二进制文件的完整路径：

    ```bash
    ~/.local/share/nvim/mason/packages/vale/vale sync
    ```

    ![vale_sync](images/vale_sync.png)

    在这种情况下，无需复制文件，因为它们将直接创建在你的 home 目录根目录中。

### :material-file-edit-outline: `lint.lua` 文件更改

还有最后一步需要完成。你需要修改位于 `~/.config/nvim/lua/configs/` 中的 `lint.lua` 文件，并添加 `vale` linter。

使用上面显示的示例，将 *vale* 添加到可用于 markdown 文件的 linter 中，需要将新的 linter 添加到已有的字符串中：

```lua
markdown = { "markdownlint", "vale" },
```

完成后，你的文件将类似于以下内容：

```lua
require("lint").linters_by_ft = {
  markdown = { "markdownlint", "vale" },
  yaml = { "yamllint" },
}

vim.api.nvim_create_autocmd({ "BufWritePost" }, {
  callback = function()
    require("lint").try_lint()
  end,
})

```

## 结论与最终思考

正常启动 `nvim` 现在将调用 `vale`，你的文档将根据你偏好的风格进行比较。打开现有文件将启动 `vale` 并显示任何标记项，而启动新文件在插入模式下不会显示任何内容。当你退出插入模式时，你的文件将被检查。这样可以避免屏幕过于杂乱。`vale` 是一个卓越的开源产品，与许多编辑器都有良好的集成。NvChad 也不例外，虽然使其运行确实需要几个步骤，但这并不是一个困难的过程。
