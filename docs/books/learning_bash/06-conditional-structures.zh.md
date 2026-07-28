---
title: Bash - 条件结构 if 和 case
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - education
  - bash scripting
  - bash
---

# Bash - 条件结构 if 和 case

****

**目标**：在本章中，您将学习如何：

:heavy_check_mark: 使用条件语法 `if`；  
:heavy_check_mark: 使用条件语法 `case`；  

:checkered_flag: **linux**, **script**, **bash**, **conditional structures**

**知识点**：:star: :star:  
**复杂度**：:star: :star: :star:  

**阅读时间**：20 分钟

****

## 条件结构

虽然 `$?` 变量用于了解测试或命令执行的结果，但它只能被显示，对脚本的执行没有影响。

但我们可以将其用于条件中。**如果**测试通过**那么**我执行此操作，**否则**我执行另一个操作。

条件分支 `if` 的语法：

```bash
if command
then
    command if $?=0
else
    command if $?!=0
fi
```

放置在 `if` 关键字之后的命令可以是任何命令，因为被评估的是其返回码（`$?`）。通常可以方便地使用 `test` 命令根据测试结果（文件存在、变量非空、写权限已设置）定义多个操作。

使用经典命令（`mkdir`、`tar`……）可以在成功时定义要执行的操作，或在失败时显示错误消息。

示例：

```bash
if [[ -e /etc/passwd ]]
then
    echo "The file exists"
else
    echo "The file does not exist"
fi

if mkdir rep
then
    cd rep
fi
```

如果 `else` 块以一个全新的 `if` 结构开始，您可以使用 `elif` 将 `else` 和 `if` 合并，如下所示：

```bash
[...]
else
  if [[ -e /etc/ ]]
[...]

[...]
# 等价于
elif [[ -e /etc ]]
[...]
```

!!! Note "总结"

    `if` / `then` / `else` / `fi` 结构评估放在 `if` 之后的命令：

    * 如果此命令的返回码是 `0`（`true`），shell 将执行放在 `then` 之后的命令；
    * 如果返回码不是 `0`（`false`），shell 将执行放在 `else` 之后的命令。

    `else` 块是可选的。

    经常需要仅在命令的评估为真时执行某些操作，而在为假时什么也不做。

    关键字 `fi` 结束该结构。

当 `then` 块中只有一个命令要执行时，可以使用更简单的语法。

如果 `$?` 为 `true` 时要执行的命令放在 `&&` 之后，而如果 `$?` 为 `false` 时要执行的命令放在 `||` 之后（可选）。

示例：

```bash
[[ -e /etc/passwd ]] && echo "The file exists" || echo "The file does not exist"
mkdir dir && echo "The directory is created".
```

也可以使用比 `if` 更轻量的结构来评估和替换变量。

此语法使用花括号实现：

* 如果变量为空，显示替换值：

    ```bash
    ${variable:-value}
    ```
  
* 如果变量非空，显示替换值：

    ```bash
    ${variable:+value}
    ```
  
* 如果变量为空，分配新值：

    ```bash
    ${variable:=value}
    ```

示例：

```bash
name=""
echo ${name:-linux}
linux
echo $name

echo ${name:=linux}
linux
echo $name
linux
echo ${name:+tux}
tux
echo $name
linux
```

!!! hint

    在决定使用 `if`、`then`、`else`、`fi` 还是使用所描述的更简单语法示例时，请考虑脚本的可读性。如果没有其他人会使用该脚本，只有您自己，那么您可以使用最适合自己的方式。如果其他人可能需要审查、调试或跟踪您创建的脚本，则要么使用更具自文档化的形式（`if`、`then` 等），要么确保彻底记录您的脚本，以便可能需要修改和使用该脚本的人真正理解更简单的语法。无论如何，记录脚本*始终*是一件好事，正如本课程前面多次指出的那样。

## 条件分支：`case` 结构

一连串的 `if` 结构可能很快变得臃肿和复杂。当涉及对同一变量的评估时，可以使用具有多个分支的条件结构。变量的值可以是指定的，或属于一个可能性列表。

**可以使用通配符**。

`case ... in` / `esac` 结构评估放在 `case` 之后的变量，并将其与定义的值进行比较。

在找到第一个相等时，执行放置在 `)` 和 `;;` 之间的命令。

被评估的变量和提议的值可以是字符串或子执行命令的结果。

放在结构末尾，`*` 选择表示要为所有之前未测试的值执行的操作。

条件分支 `case` 的语法：

```bash
case $variable in
  value1)
    commands if $variable = value1
    ;;
  value2)
    commands if $variable = value2
    ;;
  [..]
  *)
    commands for all values of $variable != of value1 and value2
    ;;
esac
```

当值可能变化时，建议使用通配符 `[]` 来指定可能性：

```bash
[Yy][Ee][Ss])
  echo "yes"
  ;;
```

字符 `|` 也允许指定一个值或另一个值：

```bash
"yes" | "YES")
  echo "yes"
  ;;
```
