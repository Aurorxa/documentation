---
title: Bash - 循环
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - education
  - bash scripting
  - bash
---

# Bash - 循环

****

**目标**：在本章中，您将学习如何：

:heavy_check_mark: 使用循环；

:checkered_flag: **linux**, **script**, **bash**, **loops**

**知识点**：:star: :star:  
**复杂度**：:star: :star: :star:  

**阅读时间**：20 分钟

****

bash shell 允许使用**循环**。这些结构允许根据静态定义的值、动态值或条件**多次执行一个命令块**（从 0 到无限次）：

* `while`
* `until`
* `for`
* `select`

无论使用哪种循环，要重复的命令都放置在**关键字** `do` 和 `done` **之间**。

## while 条件循环结构

`while` / `do` / `done` 结构评估放在 `while` 之后的命令。

如果此命令为真（`$? = 0`），则执行放在 `do` 和 `done` 之间的命令。然后脚本返回到开头再次评估该命令。

当被评估的命令为假（`$? != 0`）时，shell 在 `done` 之后的第一个命令处恢复脚本的执行。

条件循环结构 `while` 的语法：

```bash
while command
do
  command if $? = 0
done
```

使用 `while` 条件结构的示例：

```bash
while [[ -e /etc/passwd ]]
do
  echo "The file exists"
done
```

如果被评估的命令没有变化，循环将是无限的，shell 永远不会执行脚本之后放置的命令。这可能是故意的，但也可能是一个错误。因此，您必须**非常小心管理循环的命令，并找到退出循环的方法**。

要退出 `while` 循环，必须确保被评估的命令不再为真，但这并不总是可行的。

有一些命令可以更改循环的行为：

* `exit`
* `break`
* `continue`

## exit 命令

`exit` 命令结束脚本的执行。

`exit` 命令的语法：

```bash
exit [n]
```

使用 `exit` 命令的示例：

```bash
bash # to avoid being disconnected after the "exit 1
exit 1
echo $?
1
```

`exit` 命令立即结束脚本。可以通过将其作为参数给出（从 `0` 到 `255`）来指定脚本的返回码。如果没有给出参数，脚本最后一个命令的返回码将传递给 `$?` 变量。

## `break` / `continue` 命令

`break` 命令允许中断循环，跳转到 `done` 之后的第一个命令。

`continue` 命令允许重新开始循环，返回到 `done` 之后的第一个命令。

```bash
while [[ -d / ]]                                                   INT ✘  17s 
do
  echo "Do you want to continue? (yes/no)"
  read ans
  [[ $ans = "yes" ]] && continue
  [[ $ans = "no" ]] && break
done
```

## `true` / `false` 命令

`true` 命令始终返回 `true`，而 `false` 命令始终返回 `false`。

```bash
true
echo $?
0
false
echo $?
1
```

用作循环的条件时，它们允许执行无限循环或停用该循环。

示例：

```bash
while true
do
  echo "Do you want to continue? (yes/no)"
  read ans
  [[ $ans = "yes" ]] && continue
  [[ $ans = "no" ]] && break
done
```

## `until` 条件循环结构

`until` / `do` / `done` 结构评估放在 `until` 之后的命令。

如果此命令为假（`$? != 0`），则执行放在 `do` 和 `done` 之间的命令。然后脚本返回到开头再次评估该命令。

当被评估的命令为真（`$? = 0`）时，shell 在 `done` 之后的第一个命令处恢复脚本的执行。

条件循环结构 `until` 的语法：

```bash
until command
do
  command if $? != 0
done
```

使用条件结构 `until` 的示例：

```bash
until [[ -e test_until ]]
do
  echo "The file does not exist"
  touch test_until
done
```

## 备选选择结构 `select`

`select` / `do` / `done` 结构允许显示一个具有多个选择的菜单和输入请求。

列表中的每个项目都有一个编号选择。当您输入一个选择时，所选值被分配给放在 `select` 之后的变量（为此目的创建的）。

然后使用此值执行放在 `do` 和 `done` 之间的命令。

* 变量 `PS3` 包含输入选择的提示；
* 变量 `REPLY` 将返回选择的编号。

需要一个 `break` 命令来退出循环。

!!! Note

    `select` 结构对于小型简单的菜单非常有用。要自定义更完整的显示，必须在 `while` 循环中使用 `echo` 和 `read` 命令。

条件循环结构 `select` 的语法：

```bash
PS3="Your choice:"
select variable in var1 var2 var3
do
  commands
done
```

使用条件结构 `select` 的示例：

```bash
PS3="Your choice: "
select choice in coffee tea chocolate
do
  echo "You have chosen the $REPLY: $choice"
done
```

如果运行此脚本，将显示类似以下内容：

```text
1) Coffee
2) Tea
3) Chocolate
Your choice : 2
You have chosen choice 2: Tea
Your choice:
```

## 值列表的循环结构 `for`

`for` / `do` / `done` 结构将列表的第一个元素分配给放在 `for` 之后的变量（在此场合创建）。然后使用此值执行放在 `do` 和 `done` 之间的命令。然后脚本返回到开头，将列表的下一个元素分配给工作变量。当最后一个元素被使用后，shell 在 `done` 之后的第一个命令处恢复执行。

值列表循环结构 `for` 的语法：

```bash
for variable in list
do
  commands
done
```

使用条件结构 `for` 的示例：

```bash
for file in /home /etc/passwd /root/fic.txt
do
  file $file
done
```

任何产生值列表的命令都可以使用子执行放在 `in` 之后。

* 当变量 `IFS` 包含 `$' \t\n'` 时，`for` 循环将把此命令结果的**每个单词**作为要循环的元素列表。
* 当变量 `IFS` 包含 `$'\t\n'`（即没有空格）时，`for` 循环将把此命令结果的每一行作为要循环的元素。

这可以是目录中的文件。在这种情况下，变量将获取存在的文件名的每个单词作为值：

```bash
for file in $(ls -d /tmp/*)
do
  echo $file
done
```

它可以是一个文件。在这种情况下，变量将获取所浏览文件中包含的每个单词作为值，从头到尾：

```bash
cat my_file.txt
first line
second line
third line
for LINE in $(cat my_file.txt); do echo $LINE; done
first
line
second
line
third line
line
```

要逐行读取文件，必须修改 `IFS` 环境变量的值。

```bash
IFS=$'\t\n'
for LINE in $(cat my_file.txt); do echo $LINE; done
first line
second line
third line
```
