---
title: rpaste - Pastebin 工具
author: Steven Spencer
contributors:
tags:
  - rpaste
  - Mattermost
  - pastebin
---

# `rpaste` 简介

`rpaste` 是一个用于分享代码、日志输出和其他超长文本的工具。它是由 Rocky Linux 开发者创建的 pastebin。当您需要公开分享某些内容，但又不想在信息流中占据过多空间时，此工具非常有用。当使用 Mattermost（该平台已桥接其他 IRC 服务）时这个问题尤其突出。`rpaste` 工具可以安装在任何 Rocky Linux 系统上。如果您的桌面机器不是 Rocky Linux，或者您只是不想安装该工具，您也可以手动使用，直接访问 [pinnwand URL](https://rpa.st)，然后粘贴您想要分享的系统输出或文本。`rpaste` 可以让您自动创建这些信息。

## 安装

在 Rocky Linux 上安装 `rpaste`：

```bash
sudo dnf --enablerepo=extras install rpaste
```

## 用途

对于重大系统问题，您可能需要发送所有系统信息以便排查问题。为此，运行：

```bash
rpaste --sysinfo
```

这将返回 pinnwand 页面的链接：

```bash
Uploading...
Paste URL:   https://rpa.st/2GIQ
Raw URL:     https://rpa.st/raw/2GIQ
Removal URL: https://rpa.st/remove/YBWRFULDFCGTTJ4ASNLQ6UAQTA
```

然后您可以在浏览器中自行查看信息，并决定是保留还是删除并重新开始。如果想保留，可以复制 "Paste URL" 并与合作者分享，或者在 Mattermost 的信息流中分享。要删除，只需复制 "Removal URL" 并在浏览器中打开。

您可以通过管道将内容添加到 pastebin。例如，如果您想添加 3 月 10 日的 `/var/log/messages` 文件内容，可以这样做：

```bash
sudo more /var/log/messages | grep 'Mar 10' | rpaste
```

## `rpaste` 帮助

要获取命令帮助，只需输入：

```bash
rpaste --help
```

输出如下：

```bash
rpaste: A paste utility originally made for the Rocky paste service

Usage: rpaste [options] [filepath]
       command | rpaste [options]

This command can take a file or standard in as input

Options:
--life value, -x value      Sets the life time of a paste (1hour, 1day, 1week) (default: 1hour)
--type value, -t value      Sets the syntax highlighting (default: text)
--sysinfo, -s               Collects general system information (disables stdin and file input) (default: false)
--dry, -d                   Turns on dry mode, which doesn't paste the output, but shows the data to stdin (default: false)
--pastebin value, -p value  Sets the paste bin service to send to. Current supported: rpaste, fpaste (default: "rpaste")
--help, -h                  show help (default: false)
--version, -v               print the version (default: false)
```

## 结论

在处理问题、分享代码或文本时，有时需要分享大量文本。使用 `rpaste` 可以避免他人查看大量与他们无关的文本内容。这也是 Rocky Linux 聊天礼仪中的一个重要方面。
