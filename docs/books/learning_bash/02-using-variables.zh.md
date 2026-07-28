---
title: Bash - 使用变量
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - education
  - bash scripting
  - bash
---

# Bash - 使用变量

在本章中，您将学习如何在 bash 脚本中使用变量。

****

**目标**：在本章中，您将学习如何：

:heavy_check_mark: 存储信息以供后续使用；  
:heavy_check_mark: 删除和锁定变量；  
:heavy_check_mark: 使用环境变量；  
:heavy_check_mark: 替换命令；  

:checkered_flag: **linux**, **script**, **bash**, **variable**

**知识点**：:star: :star:  
**复杂度**：:star:  

**阅读时间**：10 分钟

****

## 存储信息以供后续使用

与任何编程语言一样，shell 脚本也使用变量。变量用于在内存中存储信息，以便在脚本执行过程中根据需要重复使用。

变量在接收其内容时被创建。它在脚本执行结束之前或脚本作者明确要求之前一直有效。由于脚本是从头到尾顺序执行的，因此不可能在变量创建之前调用它。

变量的内容可以在脚本运行过程中更改，因为变量会一直存在直到脚本结束。如果内容被删除，变量仍然存在但不包含任何内容。

在 shell 脚本中，变量类型的概念是可行的，但很少使用。变量的内容始终是一个字符或一个字符串。

```bash
#!/usr/bin/env bash

#
# Author : Rocky Documentation Team
# Date: March 2022
# Version 1.0.0: Save in /root the files passwd, shadow, group, and gshadow
#

# Global variables
FILE1=/etc/passwd
FILE2=/etc/shadow
FILE3=/etc/group
FILE4=/etc/gshadow

# Destination folder
DESTINATION=/root

# Clear the screen
clear

# Launch the backup
echo "Starting the backup of $FILE1, $FILE2, $FILE3, $FILE4 to $DESTINATION:"

cp $FILE1 $FILE2 $FILE3 $FILE4 $DESTINATION

echo "Backup ended!"
```

此脚本使用了变量。变量名必须以字母开头，但可以包含任意字母或数字序列。除了下划线 "_" 之外，不能使用特殊字符。

按照惯例，用户创建的变量使用小写字母命名。变量名应谨慎选择，不应过于模糊或过于复杂。然而，如果变量是一个不应被程序修改的全局变量，则可以用大写字母命名，就像本例中一样。

字符 `=` 将内容赋值给变量：

```bash
variable=value
rep_name="/home"
```

`=` 符号前后不能有空格。

一旦变量被创建，就可以通过在其前面加上美元符号 $ 来使用它。

```bash
file=file_name
touch $file
```

强烈建议用引号保护变量，如下例所示：

```bash
file=file name
touch $file
touch "$file"
```

由于变量的内容包含空格，第一个 `touch` 将创建 2 个文件，而第二个 `touch` 将创建一个文件名包含空格的文件。

要将变量名与其余文本隔离开，必须使用引号或花括号：

```bash
file=file_name
touch "$file"1
touch ${file}1
```

**建议始终使用花括号。**

使用单引号会抑制特殊字符的解析。

```bash
message="Hello"
echo "This is the content of the variable message: $message"
Here is the content of the variable message: Hello
echo 'Here is the content of the variable message: $message'
Here is the content of the variable message: $message
```

## 删除和锁定变量

`unset` 命令用于删除变量。

示例：

```bash
name="NAME"
firstname="Firstname"
echo "$name $firstname"
NAME Firstname
unset firstname
echo "$name $firstname"
NAME
```

`readonly` 或 `typeset -r` 命令用于锁定变量。

示例：

```bash
name="NAME"
readonly name
name="OTHER NAME"
bash: name: read-only variable
unset name
bash: name: read-only variable
```

!!! Note

    在脚本开头使用 `set -u`，如果使用了未声明的变量，脚本将停止执行。

## 使用环境变量

**环境变量**和**系统变量**是系统为其自身运行而使用的变量。按照惯例，这些变量使用大写字母命名。

与所有变量一样，它们可以在脚本执行时显示。即使强烈不鼓励，它们也可以被修改。

`env` 命令显示所有正在使用的环境变量。

`set` 命令显示所有正在使用的系统变量。

在众多环境变量中，有一些在 shell 脚本中使用时会很有用：

| 变量                              | 描述                                                |
|----------------------------------|-----------------------------------------------------------|
| `HOSTNAME`                       | 机器的主机名。                                 |
| `USER`、`USERNAME` 和 `LOGNAME` | 连接到会话的用户名。                |
| `PATH`                           | 查找命令的路径。                                |
| `PWD`                            | 当前目录，每次执行 cd 命令时更新。 |
| `HOME`                           | 登录目录。                                          |
| `$$`                             | 脚本执行的进程 ID。                       |
| `$?`                             | 上一次执行命令的返回码。                 |

`export` 命令用于导出一个变量。

变量仅在 shell 脚本进程的环境中有效。为了让脚本的**子进程**知道这些变量及其内容，必须导出它们。

在子进程中修改导出的变量不能回溯到父进程。

!!! note

    不带任何选项时，`export` 命令会显示环境中导出的变量的名称和值。

## 替换命令

可以将命令的结果存储在变量中。

!!! Note

    此操作仅对执行结束时返回消息的命令有效。

子执行命令的语法如下：

```bash
variable=`command`
variable=$(command) # 推荐的语法
```

示例：

```bash
day=`date +%d`
homedir=$(pwd)
```

结合我们刚刚看到的所有内容，我们的备份脚本可能如下所示：

```bash
#!/usr/bin/env bash

#
# Author : Rocky Documentation Team
# Date: March 2022
# Version 1.0.0: Save in /root the files passwd, shadow, group, and gshadow
# Version 1.0.1: Adding what we learned about variables
#

# Global variables
FILE1=/etc/passwd
FILE2=/etc/shadow
FILE3=/etc/group
FILE4=/etc/gshadow

# Destination folder
DESTINATION=/root

## Readonly variables
readonly FILE1 FILE2 FILE3 FILE4 DESTINATION

# A folder name with the day's number
dir="backup-$(date +%j)"

# Clear the screen
clear

# Launch the backup
echo "****************************************************************"
echo "     Backup Script - Backup on ${HOSTNAME}                      "
echo "****************************************************************"
echo "The backup will be made in the folder ${dir}."
echo "Creating the directory..."
mkdir -p ${DESTINATION}/${dir}

echo "Starting the backup of ${FILE1}, ${FILE2}, ${FILE3}, ${FILE4} to ${DESTINATION}/${dir}:"

cp ${FILE1} ${FILE2} ${FILE3} ${FILE4} ${DESTINATION}/${dir}

echo "Backup ended!"

# The backup is noted in the system event log:
logger "Backup of system files by ${USER} on ${HOSTNAME} in the folder ${DESTINATION}/${dir}."
```

运行我们的备份脚本：

```bash
sudo ./backup.sh
```

将得到：

```bash
****************************************************************
     Backup Script - Backup on desktop                      
****************************************************************
The backup will be made in the folder backup-088.
Creating the directory...
Starting the backup of /etc/passwd, /etc/shadow, /etc/group, /etc/gshadow to /root/backup-088:
Backup ended!
```
