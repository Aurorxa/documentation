---
title: 文档格式化指南
author: Wale Soyinka, Steven Spencer, Ezequiel Bruni
contributors: Ganna Zhyrnova, tianci li
tags:
  - contributing
  - formatting
  - documentation
---

# 文档格式化指南

## 引言

本文档提供了 Rocky Linux 文档文件内容和对项目仓库进行相关 Pull Request(PR)的指南和常规格式说明。

本指南旨在帮助所有新的贡献者能够更轻松地完成他们的首个 PR。无论您是刚开始，还是想要温习您的 Markdown 技能，我们都准备好为您提供帮助。

## 基本要求

首先要确保您的 PR 符合基本的 Markdown 语法规则。许多问题可以通过检查基本的 Markdown 来纠正，无论是在您的段落、列表、链接、URL、图片引用等方面。

### 代码检查工具(Linter)

一些贡献者希望在其开发流程中加入 Markdown 代码检查工具。本项目在文档仓库的根目录中包含了一个 [markdownlint](https://github.com/rocky-linux/documentation/blob/main/.markdownlint.yml) 配置文件，用于帮助此过程。

如果您使用的是 Visual Studio Code (VSCode)，[markdownlint 扩展](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) 是一个受欢迎的工具，可以帮助进行分析，并为您提供一些规则来遵循。您会发现，即使您不认为自己犯了任何错误，代码检查器也会在很多地方出错。这没关系。我们不考虑 Markdown 代码检查，因为我们的格式化规则在某些方面非常特殊。它只是一个帮助工具。您可以使用其中一些有用的部分，并忽略其他部分。

### HTML vs Markdown

我们在文档中尽量优先使用 Markdown 格式，而非 HTML。如果您发现某个页面包含 HTML 代码，那不是因为有人试图破坏规则，而是因为原始的 Wiki 格式被转换为 Markdown，且使用了某种必须使用 HTML 才合理的格式（比如表格），因此才出现这种情况。如果在您的 PR 审查中被指出存在 HTML 问题，不用担心。

## 页面元素

作者在创建页面时可以使用的元素包括标题、正文、代码块、列表、表格、警示框、图片和脚注。所有这些在文档中都使用 Markdown 来表示。常规规则如下：

### 页面标题

* 所有页面都必须有一个一级标题(title)。
* 标题以 H1 (标题 1) 标记，在 Markdown 中由单个 `#` 表示。
* 标题的大小写必须为每个单词首字母大写。
* 标题之后不应有其他一级标题，因为这会破坏内置的关联链接。有关详细信息，请参阅 [H1 规则说明](#h1)。

### 标题(Headings)

* 每个主要部分应使用 H2 (标题 2) 标记，在 Markdown 中由 `##` 表示。
* H2 标题的大小写必须为每个单词首字母大写。
* 子部分应使用 H3 (标题 3) 标记，在 Markdown 中由 `###` 表示。只将第一个单词的首字母大写，其余单词保持正常。（个别专有名词除外，例如 "BIND"、"MariaDB"、"Rocky Linux" 等，它们永远保持大写。）
* 标题不应再升到 H4、H5 等。如果页面内容确实需要此类标题，则可能是创建过长的页面的迹象。建议作者将其分割成多个页面。

* 示例：

    ```markdown
    ## 这部分是一个标题

    ### 这是一个子部分标题

    ### 这也是一个子部分标题
    ```

### 正文

* 正文应包裹在每行 100 个字符的限制内。许多编辑器允许基于文档重新换行。

### 代码块(Code Blocks)

Code blocks are an essential part of the documentation. All command-line entries should be marked as code. Never use a paragraph to describe a command. Code blocks can be inline (single words or short commands) or full command strings. Code blocks can apply to all languages, such as shell scripts and text configurations. Whenever possible, use the full block format with the language specified for long commands or shell scripts. Do not use preceding `$` or `#` symbols with your commands, as that interferes with the copy/paste capabilities built into the block and may cause a command or script to fail.

下面是一个代码块的示例：

```bash
dnf install my_favorite_software
```

代码块必须指定一种语言。如果代码是配置代码(conf)或文本(text)，只需在代码块中使用 "text" 或 "conf" 等通用格式。不要使用无格式的代码块。

#### 内联命令

内联命令使用单个反引号 "<code>&#96;</code>" 表示。这些内联代码用于短命令名称、文档页面以及类似的需求。

#### 键盘输入

对于需要用户从键盘输入的操作，应使用 `<kbd>KEY</kbd>` 格式。但这种使用场景并不常见。大多数情况只需使用代码块即可。

### 列表

创建列表时，仅使用 `1.` 语法作为列表序号。这有助于更容易地重新排序列表项。除非绝对必要，否则不要混用数字序号和项目符号列表。缩进用于指示包含父列表的子列表。所有缩进必须使用 4 个空格。

示例：

```markdown
1. 这是父列表的第一项。

    1. 这是子列表。

2. 这是父列表的第二项

    1. 这是第二个列表的另一项。
```

### 表格

表格很容易使用，但需要一些直观的表示。例如：

```markdown
| 软件/方法          | 版本     | 操作系统 ISO 或其他 |
|--------------------|----------|---------------------|
| `rocky-delete-old-kernels` | 全部  | Rocky Linux 全部     |
| 守护进程(Daemon)   | 9.3 及以上  | Rocky Linux 9.3+      |
```

重要的是记住编写表格时遵循以下规则：

* 标题及其后的行总是左对齐。
* 尽管标题和分隔符可以右对齐或居中，但建议保持左对齐。
* 使用[表格生成器](https://www.tablesgenerator.com/markdown_tables)等工具来帮助您创建表格。

### 警示框(Admonitions)

警示框是页面中的一个视觉元素，用来引起读者对某段信息的注意。例如：

```
!!! note
    这是一个 note 警示框的示例
```

可以使用多种类型的警示框，每种类型都有特定的目的：

| 类型        | 用途                                                                    |
|-------------|------------------------------------------------------------------------|
| note        | 引起读者对某条信息的注意                                         |
| abstract    | 总结文本                                                             |
| info        | 提供信息                                                              |
| tip         | 提供提示或建议                                                        |
| success     | 表示某事成功或已成功完成                                        |
| question    | 以问题形式提供该部分的信息                               |
| warning     | 提供警告                                                               |
| failure     | 表示某事的失败或已失败                                         |
| danger      | 表示一个危险操作                                                       |
| bug         | 表示一个已知的 bug                                                     |
| example     | 以示例形式提供该部分的信息                             |
| quote       | 提供一段引用                                                           |

### 图片

当在页面中放置图片时，理想情况下，它们应放在 `../images` 文件夹中，且该文件夹与文档页面在层级上是平级的。例如，`/guides/contribute/images/` 中的图片用于 `/guides/contribute/` 下的文档文件。图片必须使用 `alt` 标签，尽量简短描述图片内容，例如：

```text
![alt-text](../images/the_image.png)
```

如上所述，将所有内容放在一行，这样更易于处理。截图内容不应过于庞大，以免导致阅读困难。使用浏览器开发者工具来调整截图大小。

例如，如果使用 Firefox 的开发者工具，您需要按 `F12`，然后点击响应式设计模式(Responsive Design Mode)图标，或者使用 `Ctrl+Shift+M`。假设您需要截取一个 Web 页面。

在响应式设计模式打开的情况下，您可以调整高度和宽度。在获取屏幕截图的尺寸时，尽量让标识(Luminosity)小一些。以下参数是较好的示例：

`706x342`、`706x362`、`706x424`、`706x464`。

将宽度保持在 706 可以确保图片不会过大。当截图包含代码或其他难以阅读的内容时，您可以调整为更高。带有代码或文本的截图很难处理，通常应避免使用。

处理图片的推荐工作流如下：

1. 截取界面或窗口的初始截图。
2. 在浏览器中打开截图。
3. 假设您使用的是 Firefox 开发者工具，按 `F12` 启动开发者工具。
4. 调整截图尺寸。
5. 保存调整后的截图。
6. 将图片添加到文档对应的 `/images/` 目录中。
7. 使用 `![]()` 格式将图片嵌入文档。

!!! tip

    图片应为 PNG 文件。

### 脚注

脚注是提供更多页面内容而不中断主文档流的一种方式。使用脚注比插入持续打断页面流并可能使读者感到困惑的笔记更合适。一个脚注条目的示例：

```markdown
[^1] 这是一条脚注。
```

您可以使用任意数量的脚注。它们将全部按顺序排列在页面末尾。

### 大小写和特殊字符

#### 页面标题和标题

页面标题和标题遵循特定的语法规则：

* 所有页面标题和 H2 级标题都必须是标题大小写。这意味着除了 "a"、"and"、"the" 等词外，每个单词都大写。
* 对于所有 H3 级标题和更低级的标题，大写仅用于第一个单词，其他单词保持小写，除非是通常在文档中总是大写的专有名词，例如 BIND、MariaDB、Rocky Linux、DHCP 等。
* 除非必要，否则不要使用缩写。例如，用 "例如" 代替 "如"，用 "如" 代替 "如 (etc.)"。

#### 服务名称

* 在编写服务名称时，使用 `systemd` 控制的服务名称。例如：`rsyslog.service`。
* 在编写程序包名称时，使用从存储库安装时使用的名称。例如：`rsyslog`。
* 如果两者指的是同一个服务，可以在一个页面上互换使用，但通常建议使用软件包名称而不是服务名称，以帮助管理员理解安装时需要输入的内容。

### 文档内的链接

在页面内使用链接时，应尽量避免直接在段落中粘贴 URL 或有长 URL 的链接。更好的方式是使用引用链接。例如，不要使用：

```markdown
要了解如何使用 `markdownlint`，请参阅 https://github.com/DavidAnson/markdownlint
```

建议写成：

```markdown
要了解如何使用 `markdownlint`，请参阅此 [markdownlint 链接](https://github.com/DavidAnson/markdownlint)。
```

不要依赖于文档页面在标题下面的自动链接，这些链接允许方法使用 `(../contributing/#basic-requirements)` 这样的链接到一个标题，但我们的构建器在生成文档时会将它们转为数字（如 `/contributing/#_5`）。尽管这些数字链接仍然有效，但如果标题顺序发生变化，它们可能会不准确。更好的做法是使用自定义锚点(anchor)来链接到标题。下面是一个示例。

要使用自定义锚点，您需要写一个直接位于标题上方的 `<a>` 标签。例如：

```markdown
<a name="basic-requirements"></a>
## 基本要求

现在，如果您想为这个 "基本要求" 部分创建一个链接，您可以这样写：

```markdown
[基本要求](#basic-requirements)
```

请注意 "a" 标签上的 "name" 属性。它需要和 "href" 唯一一致，但以小写形式。

此方法也可以用于同一页面中的自定义锚点。进一步而言，自定义锚点是跨页面引用的更好方式。

## 结论

遵循这些格式化指南将帮助您创建既实用又与文档其余部分风格一致的文档页面，无论页面的作者是谁。
