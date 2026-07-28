---
title: sed - 搜索和替换
author: Steven Spencer
---

# `sed` - 搜索和替换

`sed` 是一个代表 "stream editor"（流编辑器）的命令。

## 约定

* `path`：实际路径。示例：`/var/www/html/`
* `filename`：实际文件名。示例：`index.php`

## 使用 `sed`

使用 `sed` 进行搜索和替换是我个人的偏好，因为您可以使用自己选择的分隔符，这使得替换包含 "/" 的 Web 链接等内容非常方便。使用 `sed` 进行原地编辑的默认示例如下：

`sed -i 's/search_for/replace_with/g' /path/filename`

但是如果搜索的字符串包含 "/" 呢？如果正斜杠是唯一可用的分隔符选项，您将不得不在搜索之前对每个正斜杠进行转义。正因为分隔符可以即时更改（无需在任何地方指定更改），`sed` 相比其他工具就有了优势。正如前面提到的，如果您在查找包含 "/" 的内容，可以通过将分隔符更改为 "|" 来实现。以下是使用此方法查找链接的示例：

`sed -i 's|search_for/with_slash|replace_string|g' /path/filename`

除反斜杠、换行符和 "s" 外，您可以使用任何单字节字符作为分隔符。例如，以下内容同样有效：

`sed -i 'sasearch_forawith_slashareplace_stringag' /path/filename`，其中 "a" 是分隔符，搜索和替换仍然有效。为了安全起见，您可以在搜索和替换时指定备份，这对于确保 `sed` 所做的更改是您*真正*想要的非常方便。这为您提供了从备份文件恢复的选项：

`sed -i.bak s|search_for|replacea_with|g /path/filename`

这将创建一个名为 `filename.bak` 的未经编辑的 `filename` 版本。

您也可以使用双引号代替单引号：

`sed -i "s|search_for/with_slash|replace_string|g" /path/filename`

## 选项说明

|选项 | 说明                                                   |
|-------|---------------------------------------------------------------|
| i     | 原地编辑文件                                            |
| i.ext | 创建一个带有指定扩展名（此处为 ext）的备份     |
| s     | 指定搜索                                              |
| g     | 指定全局替换，即替换所有匹配项 |

## 多个文件

不幸的是，`sed` 没有像 `perl` 那样的内置循环选项。要遍历多个文件，您需要在脚本中组合使用 `sed` 命令。以下是一个示例。

首先，生成一个脚本将使用的文件列表。在命令行中执行：

`find /var/www/html  -name "*.php" > phpfiles.txt`

接下来，创建一个使用 `phpfiles.txt` 的脚本：

```bash
#!/bin/bash

for file in `cat phpfiles.txt`
do
        sed -i.bak 's|search_for/with_slash|replace_string|g' $file
done
```

该脚本循环遍历 `phpfiles.txt` 中创建的所有文件，为每个文件创建备份，并全局执行搜索和替换字符串。当您确认更改符合预期后，可以删除所有备份文件。

## 其他阅读材料和示例

* `sed` [手册页](https://linux.die.net/man/1/sed)
* `sed` [更多示例](https://www.linuxtechi.com/20-sed-command-examples-linux-users/)
* `sed` & `awk` [O'Reilly 书籍](https://www.oreilly.com/library/view/sed-awk/1565922255/)

## 结论

`sed` 是一个强大的工具，在搜索和替换功能方面表现非常出色，特别是在需要灵活分隔符的场景下。
