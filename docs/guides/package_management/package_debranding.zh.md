---
title: 软件包去品牌化
---

# Rocky 软件包去品牌化操作指南

本文介绍了如何为 Rocky Linux 发行版对软件包进行去品牌化处理。

通用操作说明

首先，识别软件包中需要更改的文件。这些文件可能是文本文件、图像文件或其他类型。您可以通过深入 git.centos.org/rpms/PACKAGE/ 来识别这些文件。

为这些文件开发替代版本，但替换为 Rocky 品牌标识。根据被替换的内容类型，某些文本类型可能需要 diff/patch 文件。

替换文件放在 <https://git.rockylinux.org/patch/PACKAGE/ROCKY/_supporting/> 下
配置文件（指定如何应用补丁）放在 <https://git.rockylinux.org/patch/PACKAGE/ROCKY/CFG/*.cfg> 下

注意：使用空格而非制表符。
当 srpmproc 将软件包导入 Rocky 时，它会看到在 <https://git.rockylinux.org/patch/PACKAGE> 中所做的工作，并通过读取 ROCKY/CFG/*.cfg 下的配置文件来应用存储的去品牌化补丁。
