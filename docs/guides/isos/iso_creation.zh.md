---
title: 创建自定义 Rocky Linux ISO
author: Howard Van Der Wal
contributors: Steven Spencer, Ganna Zhyrnova
tested with: 9
tags:
- create
- custom
- ISO
- kickstart
- linux
- mkksiso
- rocky
---

**知识水平**：:star: :star:
**阅读时间**：10 分钟

## 简介

创建 ISO 的原因有很多。您可能想要修改引导过程、在安装期间添加特定软件包或更新特定的配置文件。

本指南将指导您如何构建自己的 Rocky Linux ISO。

## 前置条件

* Rocky Linux Minimal ISO 镜像（不需要 DVD 镜像）。
* 要应用到 ISO 的 `kickstart` 文件。
* 阅读 [Lorax 快速入门](https://weldr.io/lorax/lorax.html#quickstart)和 [mkksiso](https://weldr.io/lorax/mkksiso.html) 文档，熟悉如何创建 ISO。

## 软件包安装与设置

* 安装 `lorax` 软件包：

```bash
dnf install -y lorax
```

## 使用 kickstart 文件构建 ISO

* 运行 `mkksiso` 命令添加 `kickstart` 文件，然后构建新的 ISO。请注意，需要以 `root` 或具有 `sudo` 权限的用户身份运行该命令：

```bash
mkksiso --ks <PATH_TO_KICKSTART_FILE> <PATH_TO_ISO> <PATH_TO_NEW_ISO>
```

## 将仓库及其软件包添加到 ISO 镜像

* 确保要添加的仓库内部包含 `repodata` 目录。如果没有，可以使用 `createrepo_c` 命令创建，并通过 `dnf install -y createrepo_c` 安装该工具。
* 使用以下语法将仓库添加到 `kickstart` 文件：

```bash
repo --name=extra-repo --baseurl=file:///run/install/repo/<REPOSITORY>/
```

* 使用 `mkksiso` 工具的 `--add` 标志添加仓库：

```bash
mkksiso --add <LINK_TO_REPOSITORY> --ks <PATH_TO_KICKSTART_FILE> <PATH_TO_ISO> <PATH_TO_NEW_ISO>
```

* 您可以通过下面的 `baseos` 仓库示例查看此过程的更多详细信息。
* `baseos` 仓库及其所有软件包将被下载到本地：

```bash
dnf reposync -p ~ --download-metadata --repo=baseos
```

* 然后将仓库添加到 `kickstart` 文件：

```bash
repo --name=extra-repo --baseurl=file:///run/install/repo/baseos/
```

* 然后直接将 `mkksiso` 命令指向仓库目录并构建 ISO：

```bash
mkksiso --add ~/baseos --ks <PATH_TO_KICKSTART_FILE> ~/<PATH_TO_ISO> ~/<PATH_TO_NEW_ISO>
```

## 总结

一旦使用 kickstart 文件构建完成 ISO，就可以更轻松地通过单个镜像部署数百台机器，而无需逐台配置。要了解更多关于 `kickstart` 文件的信息以及多个示例，请参阅 [Kickstart Files and Rocky Linux 指南](../automation/kickstart-rocky.md)。
