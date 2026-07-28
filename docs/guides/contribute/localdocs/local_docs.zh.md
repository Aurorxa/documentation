---
title: 快速方法
author: Lukas Magauer
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.6, 9.0
tags:
  - documentation
  - local server
---

# 引言

如果您愿意，可以在本地构建文档系统，而无需使用 Docker 或 LXD。但是，如果您选择使用此流程，请注意，如果您进行大量 Python 编码或在本地使用 Python，最安全的做法是创建一个 Python 虚拟环境(Virtual Environment)，[如此处所述](https://docs.python.org/3/library/venv.html)。这将使您所有的 Python 进程相互隔离，这是推荐的做法。如果您选择使用此流程而不使用 Python 虚拟环境，请注意您承担了一定的风险。

## 流程

- 克隆 docs.rockylinux.org 仓库：

    ```bash
    git clone https://github.com/rocky-linux/docs.rockylinux.org.git
    ```

- 完成后，进入 docs.rockylinux.org 目录：

    ```bash
    cd docs.rockylinux.org
    ```

- 现在使用以下命令克隆文档仓库：

    ```bash
    git clone https://github.com/rocky-linux/documentation.git docs
    ```

- 接下来，安装 mkdocs 的 requirements.txt 文件：

    ```bash
    python3 -m pip install -r requirements.txt
    ```

- 最后运行 mkdocs 服务器：

    ```text
    mkdocs serve
    ```

## 结论

这提供了一种快速简单的方法来运行文档的本地副本，而无需 Docker 或 LXD。如果您选择此方法，您应该真正设置一个 Python 虚拟环境来保护您的其他 Python 进程。
