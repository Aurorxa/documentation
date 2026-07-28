---
title: RockyDocs 脚本方法
author: Steven Spencer
contributors: Ganna Zhyrnova
tested_with: 8.6, 9.0
tags:
  - contribute
  - local docs
  - code
  - script
---

## 引言

本节介绍一个名为 `RockyDocs` 的脚本，用于在本地工作站上快速轻松地构建和测试文档站点。这种方法适用于那些想要比[快速方法](local_docs.md)更强大的功能，又觉得[Python VENV 方法](mkdocs_venv.md)过于复杂，同时还想避免运行[基于 LXD 的完整配置](mkdocs_lsyncd.md)的用户。`RockyDocs` 脚本使用 Docker 在后台运行 `mkdocs`，而您在前端工作。它易用，镜像体积小。

如果这听起来像您需要的，那么让我们配置它。

## 前提条件

您需要安装并运行 Docker。请参阅 [Docker 安装文档](../../containers/docker.md)，了解如何在您的系统上安装 Docker。

您还需要一个 GitHub 账号和 _Rocky Linux 文档_仓库的一个 fork（要贡献内容，您必须先 fork [Rocky Linux 文档仓库](https://github.com/rocky-linux/documentation)）。

## 安装 RockyDocs

要安装，请执行以下步骤。

首先，确保您位于主目录：

```bash
cd ~
```

克隆 RockyDocs 仓库。该仓库包含脚本以及所需的资源文件：

```bash
git clone https://github.com/tcooper/rockydocs.git
```

RockyDocs 的主页位于 [RockyDocs 主页](https://github.com/tcooper/rockydocs/)。

脚本构建 RockyDocs Docker 容器并配置文档构建环境。请运行以下命令：

```bash
./rockydocs/rockydocs setup
```

## 导入您的文档

`rockydocs setup` 完成后，目录中包含 `./documents/` 子目录，该子目录指向您之前创建的 _documentation_ fork。进入文档目录：

```bash
cd ./rockydocs/documentation
```

添加一个新的 `git remote` 指向您的 fork。

首先，重命名克隆中的 `origin` 远程为 `upstream`：

```bash
git remote rename origin upstream
```

现在，使用以下命令正确设置 `origin` 远程。将 "username" 替换为您的实际用户名：

```bash
git remote add origin https://github.com/username/documentation.git
```

允许 `rockydocs` 跟踪您的文档分支。

例如，如果您有一个名为 `pr-1` 的文档分支，您可以通过访问 `http://localhost:8000/pr-1/` 来预览它。

您可以通过执行以下命令来跟踪文档分支：

```bash
./rockydocs/rockydocs track pr-1
```

## 使用 RockyDocs

### 构建文档站点

您可以使用以下命令构建（或重建）文档站点：

```bash
./rockydocs/rockydocs build
```

这将构建文档站点。请注意，当您开始跟踪一个新分支时，您需要先运行 `rockydocs build`，然后才能在 Web 浏览器中看到更改。之后，分支中的更改将被实时跟踪。

### `rockydocs` 命令概览

`rockydocs` 命令接受单个参数，例如：

```bash
./rockydocs/rockydocs build
```

| 命令     | 用途                                                                                                                                 |
|----------|--------------------------------------------------------------------------------------------------------------------------------------|
| `setup`  | 首次使用时运行。构建 Docker 镜像，并设置 `documentation` 目录。运行 `setup` 后，您需要进入 `documentation` 目录，将其 `origin` 远程切换为指向您的 GitHub fork。 |
| `track`  | 跟踪一个文档分支。用户通过 Web 浏览器访问 `http://localhost:8000/###/` 可以浏览该分支，其中 `###` 是分支名称。                                      |
| `build`  | 构建文档站点（或重建，因为您进行了一些更改）。在 Web 浏览器中查看更改之前，您需要运行此命令。                                                    |
| `ls`     | 列出所有当前被跟踪的文档分支。                                                                                                         |
| `rm`     | 停止跟踪一个文档分支。自动移除 `documents/build.d/` 中的文件。                                                                          |
| `shell`  | 进入正在运行的 rockydocs 容器内部。                                                                                                     |
| `update` | 更新 rockydocs 本地仓库到 master 分支的最新版本。                                                                                     |

## 结论

`rockydocs` 脚本使查看正在进行的文档更改变得快速简便。对于那些熟悉容器且不想被其他选项的复杂性所困扰的人来说，这是理想的选择。
