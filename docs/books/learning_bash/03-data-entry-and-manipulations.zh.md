---
title: Bash - 数据输入与操作
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - education
  - bash scripting
  - bash
---

# Bash - 数据输入与操作

在本章中，您将学习如何让脚本与用户交互并操作数据。

****

**目标**：在本章中，您将学习如何：

:heavy_check_mark: 读取用户输入；  
:heavy_check_mark: 操作数据条目；  
:heavy_check_mark: 在脚本中使用参数；  
:heavy_check_mark: 管理位置变量；  

:checkered_flag: **linux**, **script**, **bash**, **variable**  

**知识点**：:star: :star:  
**复杂度**：:star: :star:  

**阅读时间**：10 分钟

****

根据脚本的用途，它可能需要在启动时或执行过程中获取信息。这些信息不是在编写脚本时预先确定的，可以来自文件、用户输入，或者在输入脚本命令时作为参数传递，类似于许多 Linux 命令。

## `read` 命令

`read` 命令允许您输入一个字符串并将其存储在变量中。

`read` 命令的语法：

```bash
read [-n X] [-p] [-s] [variable]
```

下面的第一个示例提示您输入两个变量："name" 和 "firstname"，但由于没有提示信息，您需要事先知道这一点。在这种特定的输入情况下，每个变量输入之间需要用空格分隔。第二个示例提示输入变量 "name" 并包含提示文本：

```bash
read name firstname
read -p "Please type your name: " name
```

| 选项 | 功能                                     |
|--------|-----------------------------------------------|
| `-p`   | 显示提示消息。                    |
| `-n`   | 限制输入的字符数。 |
| `-s`   | 隐藏输入。                              |

当使用 `-n` 选项时，shell 会在输入指定数量的字符后自动验证输入。用户无需按下 ++enter++ 键。

```bash
read -n5 name
```

`read` 命令允许在用户输入信息时中断脚本的执行。用户的输入被分解为单词，分配给一个或多个预定义的变量。单词是由字段分隔符分隔的字符串。

输入的结束通过按下 ++enter++ 键来确定。

一旦输入被验证，每个单词将被存储在预定义的变量中。

单词的分割由字段分隔符字符定义。此分隔符存储在系统变量 `IFS`（**Internal Field Separator**，内部字段分隔符）中。

```bash
set | grep IFS
IFS=$' \t\n'
```

默认情况下，IFS 包含空格、制表符和换行符。

当不指定变量使用时，此命令只是暂停脚本。脚本在输入被验证后继续执行。

这用于在调试时暂停脚本，或提示用户按 ++enter++ 继续。

```bash
echo -n "Press [ENTER] to continue..."
read
```

## `cut` 命令

`cut` 命令用于在文件或流中隔离一列。

`cut` 命令的语法：

```bash
cut [-cx] [-dy] [-fz] file
```

`cut` 命令的使用示例：

```bash
cut -d: -f1 /etc/passwd
```

| 选项 | 说明                                                      |
|--------|------------------------------------------------------------------|
| `-c`   | 指定要选择的字符序列号。 |
| `-d`   | 指定字段分隔符。                                   |
| `-f`   | 指定要选择的列的顺序号。             |

此命令的主要优势在于它与流的结合使用，例如与 `grep` 命令和 `|` 管道的结合。

* `grep` 命令是"垂直"工作的（从文件的所有行中隔离出一行）。
* 两个命令的组合允许**隔离文件中的特定字段**。

示例：

```bash
grep "^root:" /etc/passwd | cut -d: -f3
0
```

!!! NOTE

    使用相同字段分隔符的单一结构配置文件是这种命令组合的理想目标。

## `tr` 命令

`tr` 命令用于转换字符串。

`tr` 命令的语法：

```bash
tr [-csd] string1 string2
```

| 选项 | 说明                                                                                            |
|--------|--------------------------------------------------------------------------------------------------------|
| `-c`   | 将所有未在第一个字符串中指定的字符转换为第二个字符串的字符。 |
| `-d`   | 删除指定的字符。                                                                       |
| `-s`   | 将指定字符缩减为单个单元。                                                       |

以下是一个使用 `tr` 命令的示例。如果您使用 `grep` 返回 root 的 `passwd` 文件条目，您会得到：

```bash
grep root /etc/passwd
```

返回：

```bash
root:x:0:0:root:/root:/bin/bash
```

现在让我们使用 `tr` 命令并缩减行中的 "o"：

```bash
grep root /etc/passwd | tr -s "o"
```

返回结果：

```bash
rot:x:0:0:rot:/rot:/bin/bash
```

## 提取文件的名称和路径

`basename` 命令用于从路径中提取文件名。

`dirname` 命令用于提取文件的父路径。

示例：

```bash
echo $FILE=/usr/bin/passwd
basename $FILE
```

结果为 "passwd"

```bash
dirname $FILE
```

结果为 "/usr/bin"

## 脚本的参数

使用 `read` 命令请求输入信息会在用户未输入任何信息时中断脚本的执行。

这种方法虽然非常用户友好，但如果脚本计划在夜间运行则有其局限性。为了克服这个问题，可以通过参数注入所需的信息。

许多 Linux 命令都是基于此原理工作的。

这种方式的优点是，一旦脚本执行，就不需要任何人工干预即可完成。

其主要缺点是用户必须了解脚本的语法以避免错误。

参数在输入脚本命令时填写。它们之间用空格分隔。

```bash
./script argument1 argument2
```

一旦执行，脚本将输入的参数保存在预定义的变量中：**位置变量**。

这些变量可以在脚本中像其他变量一样使用，但它们不能被赋值。

* 未使用的位置变量存在但是空的。
* 位置变量始终以相同的方式定义：

| 变量          | 说明                                             |
|--------------|---------------------------------------------------------|
| `$0`         | 包含所输入的脚本名称。             |
| `$1` 至 `$9` | 包含第 1 到第 9 个参数的值           |
| `${x}`       | 包含参数 `x` 的值，x 大于 9。 |
| `$#`         | 包含传递的参数数量。                |
| `$*` 或 `$@` | 在一个变量中包含所有传递的参数。      |

示例：

```bash
#!/usr/bin/env bash
#
# Author : Damien dit LeDub
# Date : september 2019
# Version 1.0.0 : Display the value of the positional arguments
# From 1 to 3

# The field separator will be "," or space
# Important to see the difference in $* and $@
IFS=", "

# Display a text on the screen:
echo "The number of arguments (\$#) = $#"
echo "The name of the script  (\$0) = $0"
echo "The 1st argument        (\$1) = $1"
echo "The 2nd argument        (\$2) = $2"
echo "The 3rd argument        (\$3) = $3"
echo "All separated by IFS    (\$*) = $*"
echo "All without separation  (\$@) = $@"
```

将得到：

```bash
$ ./arguments.sh one two "tree four"
The number of arguments ($#) = 3
The name of the script  ($0) = ./arguments.sh
The 1st argument        ($1) = one
The 2nd argument        ($2) = two
The 3rd argument        ($3) = tree four
All separated by IFS    ($*) = one,two,tree four
All without separation  ($@) = one two tree four
```

!!! warning

    注意 `$@` 和 `$*` 之间的区别。区别在于参数存储格式：

    * `$*`：以 `"$1 $2 $3 ..."` 格式包含参数
    * `$@`：以 `"$1" "$2" "$3" ...` 格式包含参数

    通过修改 `IFS` 环境变量可以看到这种差异。

### `shift` 命令

`shift` 命令用于移动位置变量。

让我们修改前面的示例来说明 `shift` 命令对位置变量的影响：

```bash
#!/usr/bin/env bash
#
# Author : Damien dit LeDub
# Date : september 2019
# Version 1.0.0 : Display the value of the positional arguments
# From 1 to 3

# The field separator will be "," or space
# Important to see the difference in $* and $@
IFS=", "

# Display a text on the screen:
echo "The number of arguments (\$#) = $#"
echo "The 1st argument        (\$1) = $1"
echo "The 2nd argument        (\$2) = $2"
echo "The 3rd argument        (\$3) = $3"
echo "All separated by IFS    (\$*) = $*"
echo "All without separation  (\$@) = $@"

shift 2
echo ""
echo "-------- SHIFT 2 ----------------"
echo ""

echo "The number of arguments (\$#) = $#"
echo "The 1st argument        (\$1) = $1"
echo "The 2nd argument        (\$2) = $2"
echo "The 3rd argument        (\$3) = $3"
echo "All separated by IFS    (\$*) = $*"
echo "All without separation  (\$@) = $@"
```

将得到：

```bash
./arguments.sh one two "tree four"
The number of arguments ($#) = 3
The 1st argument        ($1) = one
The 2nd argument        ($2) = two
The 3rd argument        ($3) = tree four
All separated by IFS    ($*) = one,two,tree four
All without separation  ($@) = one two tree four

-------- SHIFT 2 ----------------

The number of arguments ($#) = 1
The 1st argument        ($1) = tree four
The 2nd argument        ($2) =
The 3rd argument        ($3) =
All separated by IFS    ($*) = tree four
All without separation  ($@) = tree four
```

正如您所看到的，`shift` 命令将参数的位置"向左"移动，移除了前 2 个。

!!! WARNING

    使用 `shift` 命令时，`$#` 和 `$*` 变量会相应地修改。

### `set` 命令

`set` 命令将字符串拆分为位置变量。

`set` 命令的语法：

```bash
set [value] [$variable]
```

示例：

```bash
$ set one two three
$ echo $1 $2 $3 $#
one two three 3
$ variable="four five six"
$ set $variable
$ echo $1 $2 $3 $#
four five six 3
```

现在您可以像之前看到的那样使用位置变量了。
