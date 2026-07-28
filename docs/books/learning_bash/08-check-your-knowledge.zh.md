---
title: Bash - 检验您的知识
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - education
  - bash scripting
  - bash
---

# Bash - 检验您的知识

:heavy_check_mark: 每个命令在执行结束时都必须返回一个返回码：

- [ ] True
- [ ] False

:heavy_check_mark: 返回码 0 表示执行错误：

- [ ] True
- [ ] False

:heavy_check_mark: 返回码存储在变量 `$@` 中：

- [ ] True
- [ ] False

:heavy_check_mark: `test` 命令允许您：

- [ ] 测试文件的类型
- [ ] 测试变量
- [ ] 比较数字
- [ ] 比较 2 个文件的内容

:heavy_check_mark: `expr` 命令：

- [ ] 连接 2 个字符串
- [ ] 执行数学运算
- [ ] 在屏幕上显示文本

:heavy_check_mark: 下面的条件结构语法对您来说是否看起来正确？请解释原因。

```bash
if command
    command if $?=0
else
    command if $?!=0
fi
```

- [ ] True
- [ ] False

:heavy_check_mark: 以下语法表示什么：`${variable:=value}`

- [ ] 如果变量为空，显示替换值
- [ ] 如果变量非空，显示替换值
- [ ] 如果变量为空，分配新值

:heavy_check_mark: 下面的备选条件结构语法对您来说是否看起来正确？请解释原因。

```bash
case $variable in
  value1)
    commands if $variable = value1
  value2)
    commands if $variable = value2
  *)
    commands for all values of $variable != of value1 and value2
    ;;
esac
```

- [ ] True
- [ ] False

:heavy_check_mark: 以下哪一项不是循环结构？

- [ ] while
- [ ] until
- [ ] loop
- [ ] for
