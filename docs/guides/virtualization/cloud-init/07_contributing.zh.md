---
title: 7. 贡献
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - cloud-init
  - open source
  - development
  - python
---

## 为 cloud-init 项目做出贡献

恭喜你！你已经从 `cloud-init` 的基本概念一路走到高级配置和故障排除技术。你现在是一名 `cloud-init` 高级用户。这最后一章为你开启旅程的下一步大门：从 `cloud-init` 的消费者转变为潜在的贡献者。

`cloud-init` 是一个关键的开源项目，社区贡献使它蓬勃发展。无论是修复文档中的一个拼写错误、报告一个 bug，还是编写一个全新的模块，每一个贡献都有帮助。本章提供了一份高层次的地图，用于理解源代码、构建自定义模块以及与上游社区互动。它不是一份详尽的开发者指南，而是一个友好的入门介绍，让你参与进来。

## 1. `cloud-init` 源代码概览

在你能做出贡献之前，请先熟悉一下项目。让我们探索源代码并设置一个基本的开发环境。

### 编程语言与代码仓库

`cloud-init` 几乎完全用 **Python** 编写，Canonical 在其官方 **GitHub** 镜像上托管源代码仓库。

* **GitHub 镜像：** [https://github.com/canonical/cloud-init](https://github.com/canonical/cloud-init)

要获取源代码，你可以克隆 GitHub 仓库：

```bash
# 将源代码克隆到你的本地机器
git clone https://github.com/canonical/cloud-init.git
cd cloud-init
```

### 设置开发环境

为了在不影响系统 Python 软件包的情况下编写代码，务必使用虚拟环境 (virtual environment)。

```bash
# 创建一个 Python 虚拟环境
python3 -m venv .venv

# 激活虚拟环境
source .venv/bin/activate

# 安装所需的开发依赖项
pip install -r requirements-dev.txt
```

### 源代码概览

浏览一个新的代码库可能会让人望而生畏。以下是几个最重要的目录：

* `cloudinit/`：这是主要的 Python 源代码目录。
* `cloudinit/sources/`：此目录包含 **数据源** (Datasources) 的代码（例如 `DataSourceNoCloud.py`）。这是 `cloud-init` 检测并从不同云平台读取配置的方式。
* `cloudinit/config/`：这是 **模块** (Modules) 所在的位置（例如 `cc_packages.py`、`cc_users_groups.py`）。`cc_` 前缀是 `#cloud-config` 启用的模块的一个约定。这是最常见的新功能贡献位置。
* `doc/`：项目的官方文档。改进文档是做出首次贡献的最佳方式之一。
* `tests/`：项目的综合测试套件。

## 2. 编写一个基本的自定义模块

虽然 `runcmd` 很有用，但编写一个合适的模块是创建可重用、可移植且幂等的配置的最佳方式。

让我们创建最简单的模块：一个从 `user-data` 中读取配置键并向 `cloud-init` 日志写入一条消息的模块。

1. **创建模块文件：** 创建一个名为 `cloudinit/config/cc_hello_world.py` 的新文件。

    ```python
    # Filename: cloudinit/config/cc_hello_world.py

    # 此模块运行的频率和阶段的列表
    frequency = "once-per-instance"
    distros = ["all"]

    def handle(name, cfg, cloud, log, args):
        # 从 user-data 配置中获取 'message' 键。
        # 如果不存在，使用默认值。
        message = cfg.get("message", "Hello from a custom module!")

        # 将消息写入主要的 cloud-init 日志。
        log.info(f"Hello World Module says: {message}")
    ```

2. **启用模块：** 仅仅创建文件是不够的。你必须告诉 `cloud-init` 运行它。在 `/etc/cloud/cloud.cfg.d/99-my-modules.cfg` 创建一个文件，并将你的模块添加到模块列表中：

    ```yaml
    # 将我们的自定义模块添加到在 config 阶段运行的模块列表中
    cloud_config_modules:
      - hello_world
    ```

3. **使用模块：** 现在，你可以在 `user-data` 中使用该模块。顶级键 (`hello_world`) 应与不带 `cc_` 前缀的模块名称匹配。

    ```yaml
    #cloud-config
    hello_world:
      message: "My first custom module is working!"
    ```

使用此配置启动虚拟机后，检查 `/var/log/cloud-init.log` 以查找你的自定义消息，确认你的模块已工作。

## 3. 贡献工作流

为开源项目做贡献遵循一个标准工作流。以下是简化的概述：

1. **寻找要处理的事情：** 最好的起点是项目在 Launchpad 上的问题追踪器。请务必寻找 bug 或功能请求。鼓励新手从文档修复或被标记为 "low-hanging-fruit" (容易摘的果子) 或 "good first issue" (好的入门问题) 的问题开始。

2. **Fork 并创建分支：** 在 GitHub 上创建你自己的 `cloud-init` 仓库副本（一个 "fork"）。然后，为你的更改创建一个新的分支。

    ```bash
    git checkout -b my-documentation-fix
    ```

3. **做出更改并提交：** 进行你的代码或文档更改。提交时，写一条清晰的消息描述你做了什么。`-s` 标志添加一行 `Signed-off-by` (签署者)，证明你编写了补丁或有权贡献它。

    ```bash
    git commit -s -m "Doc: Fix typo in the users module documentation"
    ```

4. **包含测试：** 所有重要的贡献，特别是新功能，都必须包含测试。探索 `tests/` 目录以了解现有模块是如何测试的。

5. **提交 Pull Request (PR)：** 将你的分支推送到你在 GitHub 上的 fork，并向 `canonical/cloud-init` 仓库的 `main` 分支发起 Pull Request。这是你正式请求将你的工作包含在项目中的过程。

6. **参与代码审查：** 项目维护者将审查你的 PR。他们可能会提出问题或请求更改。这是一个协作的过程。参与反馈讨论是开源贡献的关键部分。

### 社区互动

要了解更多、提出问题并与社区互动，你可以加入 OFTC IRC 网络上的 `#cloud-init` 频道或官方邮件列表。

## 最后的寄语

恭喜你完成本指南。你已经从一名新手成长为 `cloud-init` 高级用户，现在你有一张地图指引你进入开源贡献的世界。`cloud-init` 社区是热情的，重视你的贡献，无论多小。快乐构建！
