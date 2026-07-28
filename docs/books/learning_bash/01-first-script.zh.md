---
title: Bash - 第一个脚本
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - education
  - bash scripting
  - bash
---

# Bash - 第一个脚本

在本章中，您将学习如何编写您的第一个 bash 脚本。

****

**目标**：在本章中，您将学习如何：

:heavy_check_mark: 编写您的第一个 bash 脚本；  
:heavy_check_mark: 执行您的第一个脚本；  
:heavy_check_mark: 使用所谓的 shebang 指定要使用的 shell；  

:checkered_flag: **linux**, **script**, **bash**

**知识点**：:star:  
**复杂度**：:star:  

**阅读时间**：10 分钟

****

## 我的第一个脚本

开始编写 shell 脚本时，使用支持语法高亮的文本编辑器会很方便。

例如，`vim` 就是一个很好的工具。

脚本的名称应遵守一些规则：

* 不能使用已有命令的名称；
* 只能使用字母数字字符，即不能有带重音的字符或空格；
* 扩展名 .sh 表示它是一个 shell 脚本。

!!! note

    作者在这些课程中使用 "$" 来表示用户的命令提示符。

```bash
#!/usr/bin/env bash
#
# Author : Rocky Documentation Team
# Date: March 2022
# Version 1.0.0: Displays the text "Hello world!"
#

# Displays a text on the screen :
echo "Hello world!"
```

要运行此脚本，可以将其作为 bash 的参数：

```bash
$ bash hello-world.sh
Hello world !
```

或者，更简单的方式是在赋予执行权限后运行：

```bash
$ chmod u+x ./hello-world.sh
$ ./hello-world.sh
Hello world !
```

!!! note

    当您在脚本所在的目录中时，需要在脚本名称前加上 `./` 来调用它。如果不在该目录中，则需要使用脚本的完整路径来调用它，或者将其放置在 PATH 环境变量中的目录中：（例如：`/usr/local/sbin`、`/usr/local/bin` 等。）
    解释器会拒绝执行当前目录中未指定路径（此处为前面的 `./`）的脚本。

    `chmod` 命令只需在新创建的脚本上执行一次。

任何脚本的第一行都要指明用于执行它的 shell 二进制文件的名称。如果您想使用 `ksh` shell 或解释型语言 `python`，则需要替换该行：

```bash
#!/usr/bin/env bash
```

替换为：

```bash
#!/usr/bin/env ksh
```

或替换为：

```bash
#!/usr/bin/env python
```

这第一行被称为 `shebang`（释伴）。它以字符 `#!` 开头，后跟要使用的命令解释器二进制文件的路径。

!!! hint "关于 shebang"

    您可能在一些看过的脚本中遇到不包含 "env" 部分、只包含要使用的解释器的 "shebang"。（例如：`#!/bin/bash`）。作者的方法被认为是推荐且正确的 "shebang" 格式。

    为什么推荐作者的方法？因为它提高了脚本的可移植性。如果由于某种原因解释器位于一个完全不同的目录中，使用作者的方法仍然**可以**找到解释器。

在整个编写过程中，您应该考虑审阅脚本，特别是使用注释：

* 在开头做一个总体说明，指明脚本的用途、作者、版本、使用方法等。
* 在正文中帮助理解各个操作。

注释可以放在单独的一行，也可以放在包含命令的行末尾。

示例：

```bash
# This program displays the date
date # This line is the line that displays the date!
```
