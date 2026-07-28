---
title: Python VENV 方法
author: Steven Spencer
contributors: Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - contribute
  - local docs
  - code
---

# 引言

本文档描述了如何使用 Python 虚拟环境(Python Virtual Environment)运行 `mkdocs` 的实例。这是[快速方法](local_docs.md)的修订版。它使用 `requirements.txt` 执行安装，然后使用 Python 创建 VENV。此版本也是[专家贡献指南](../expert_contributing.md)的一部分。

本指南的其余部分假设您是 root 用户，或可以使用 `sudo`，并且具备必要的先决条件。

## 前提条件

运行此环境所需的前提条件：

* 安装了 Python3（根据您的操作系统版本，可能需要使用`dnf install python3`）。
* 安装了 `pip3`（根据您的操作系统版本，可能需要使用 `dnf install python3-pip`)。
* Git（使用 `dnf install git`）。
* [使用公钥的 SSH 设置](../../security/ssh_public_private_keys.md)
* Python `venv` 模块（`python3-venv` 软件包）。根据您的操作系统，可能需要使用 `dnf install python3-venv`。

## 首次设置（假设您已按先决条件中的描述安装了需求）

以下步骤假设您要从头开始执行过程。

- 克隆 docs.rockylinux.org 仓库：

    ```bash
    git clone git@github.com:rocky-linux/docs.rockylinux.org.git
    ```

- 完成后，进入 docs.rockylinux.org 目录：

    ```bash
    cd docs.rockylinux.org
    ```

- 确保您在存储库中当前位于正确的分支。目前仓库使用 `main` 分支。运行以下命令查看您在仓库中的位置：

    ```bash
    git branch
    ```

    输出应显示当前分支带有星号，如下所示：

    ```bash
    * main  
    ```

    在现代 Git 版本中，输出可能显示以下内容：

    ```bash
    * (main)
    ```

- 现在创建 Python VENV（您会注意到，在您安装的 Python 版本中首次使用时，这不仅会创建 VENV，还会更新 `pip`）：

    ```bash
    python3 -m venv venv
    ```

- 激活 VENV：

    ```bash
    source venv/bin/activate
    ```

    您会注意到提示符发生了变化，显示您正在 VENV（虚拟环境）中：

    ```bash
    (venv) [rocky_pc docs.rockylinux.org]$
    ```

- 接下来，安装 `requirements.txt` 文件中的要求：

    ```bash
    pip install -r requirements.txt
    ```

    这将成功安装所有必要模块。

- 现在，使用以下命令克隆 Rocky Linux 文档仓库：

    ```bash
    git clone git@github.com:rocky-linux/documentation.git docs
    ```

    如果您先阅读了版本管理说明，您可以克隆仓库并使用 git 远程名称。如果您遵循[早期方法](mkdocs_lsyncd.md)，请按照该文档中的说明设置 git 远程。

    如果不确定，只需使用此处的克隆方法。

    现在尝试构建和提供文档：

    ```bash
    mkdocs serve
    ```

- 假设一切正常，您将看到一则免责声明，说明不打算将其用作生产网络服务器，还有大量调试信息，最后是一行通知，简单说明此实例在 localhost:8000 上提供服务。要查看文档，您需要打开 Web 浏览器并输入以下 URL：

    ```text
    http://localhost:8000
    ```

    如果一切正常，您应该看到文档站点的本地实例。

    - 注意：完成后，您可以简单地删除 docs.rockylinux.org 目录及其所有内容，所有这些就会消失。您的计算机上不会留下任何软件包或垃圾残留。

## 版本管理说明

您所在的文档分支将决定您所看到的内容。换句话说，虽然此方法有效，但如果不进行额外修改，它无法区分文档版本（8、9 和主版本/10）。当您克隆文档仓库（上面的 `docs` 目录）时，是使用 `main` 分支完成的。要查看其他版本，您需要在 `docs/` 目录中执行 `git checkout`：

```text
cd docs
git checkout rocky-8
```

或：

```text
cd docs
git checkout rocky-9
```

之后，再次运行 `mkdocs serve` 就会显示另一个版本（9 或 8）的文档。请注意，以这种方式切换版本会中断您通过 `mkdocs` 运行的内容。您需要取消当前运行的 `mkdocs serve` 实例，并在检出另一个版本后重新启动。

如果您使用版本管理方法进行存储库克隆，则这是多余的，因为所描述的整个方法依赖于一个不同的过程来维护 Rocky Linux 和您自己的 fork。当时机成熟，评审您自己的代码时，可能需要合并两个流程或选择最适合您的方法。阅读[专家贡献指南](../expert_contributing.md)之后再决定如何继续。

## 结论

使用 VENV 来运行文档本地副本是一种不错的折中方案。代码是隔离的，不必依赖容器技术，对本地操作系统的影响极小。VENV 用完即弃，易于清理。请注意，这不是保留一种或多种版本感知文档本地副本的完整方法。它是一个很好的替代方案，可以在提交修改之前快速进行本地化检查。
