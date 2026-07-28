---
title: 测试 CPU 兼容性
author: Steven Spencer
contributors: Louis Abel, Ganna Zhyrnova
tags:
  - cpu test 
---

# 简介

在 x86-64 平台上，某些安装可能导致内核恐慌 (kernel panic)。在大多数情况下，==这是由于 CPU 与 Rocky Linux 不兼容==。

## 测试

1. 获取 Rocky Linux 9、Fedora 或其他发行版的引导镜像。

2. 在要安装 Rocky Linux 10 的机器上启动此 Live 镜像。

3. 启动完成后，打开终端窗口并运行以下过程：

    ```bash
    /lib64/ld-linux-x86-64.so.2 --help | grep x86-64
    ```

    您应该收到类似以下内容的输出：

    ```bash
    Usage: /lib64/ld-linux-x86-64.so.2 [OPTION]... EXECUTABLE-FILE [ARGS-FOR-PROGRAM...]
    This program interpreter self-identifies as: /lib64/ld-linux-x86-64.so.2
    x86-64-v4
    x86-64-v3 (supported, searched)
    x86-64-v2 (supported, searched)
    ```

    此输出表示最低所需的 x86-64 版本（v3）。在这种情况下，可以继续安装。如果 "x86-64-v3" 条目旁边缺少 "(supported, searched)"，则您的 CPU **不**与 Rocky Linux 10 兼容。如果测试表明您的安装可以继续，并且还列出了 x86-64-v4 为 "(supported, searched)"，则您的 CPU 对 Rocky Linux 未来版本具有良好支持。
