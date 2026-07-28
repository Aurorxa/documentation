---
title: 设置本地 Rocky 仓库
author: codedude
contributors: Steven Spencer
update: 09-Dec-2021
---

# 简介

有时您需要为本地的虚拟机、实验环境等提供 Rocky 仓库。如果带宽是一个问题，这也可以帮助节省带宽。本文将引导您使用 `rsync` 将 Rocky 仓库复制到本地 Web 服务器。搭建 Web 服务器超出了这篇简短文章的范围。

## 要求

* 一个 Web 服务器

## 代码

```bash
#!/bin/bash
repos_base_dir="/web/path"

# Start sync if base repo directory exist
if [[ -d "$repos_base_dir" ]] ; then
  # Start Sync
  rsync  -avSHP --progress --delete --exclude-from=/opt/scripts/excludes.txt rsync://ord.mirror.rackspace.com/rocky  "$repos_base_dir" --delete-excluded
  # Download Rocky 8 repository key
  if [[ -e /web/path/RPM-GPG-KEY-rockyofficial ]]; then
     exit
  else
      wget -P $repos_base_dir https://dl.rockylinux.org/pub/rocky/RPM-GPG-KEY-rockyofficial
  fi
fi
```

## 分解说明

这个简单的 shell 脚本使用 `rsync` 从最近的镜像拉取仓库文件。它还使用了 "exclude"（排除）选项，该选项在文本文件中以不应包含的关键字形式定义。如果磁盘空间有限，或者出于某种原因不想要所有内容，排除选项非常有用。我们可以使用 `*` 作为通配符。使用 `*/ng` 时要小心，因为它会排除任何匹配这些字符的内容。示例如下：

```bash
*/source*
*/debug*
*/images*
*/Devel*
8/*
8.4-RC1/*
8.4-RC1
```

## 结尾

一个简单的脚本，可以帮助节省带宽或使搭建实验环境变得稍微容易一些。
