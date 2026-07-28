---
title: Podman 方法
author: Steven Spencer
contributors: tianci li, Ganna Zhyrnova, Connor Francis, Colussi Franco
tested_with: 8.10, 9.4
tags:
  - contribute
  - local docs
  - code
  - podman
---

# RockyDocs webdev (v2) — 基于 Podman 的本地文档环境

## 引言

罗列使用选项，却没有达成共识，这并非理想的做法。我是 Steven Spencer，我亲笔写下这段引言，是为了告诉你，其他方法都并非首选。Rocky Docs WebDev v2（或称 "localdocs"）**才是**构建和处理文档的正确方式。截至本文撰写时，我们正处于实现这一方法的早期阶段。但随着时间的推移，我们将加强这一体验，并将其设定为核心流程。

如果你现在正在使用其中一种其他方法，请不要慌张。目前每一种方法都能正常工作。然而，如果你正在寻找一种**新**的方式来构建和预览本地文档，那么请继续阅读。同时，我们也乐于听取你的反馈意见，以改进这一方法，使其变得更加便捷高效。

## 目标

我们的目标是为所有人提供更好的本地文档使用体验。为了实现这一目标，我们使用 Podman 容器，这有助于提供以下功能：

1. 导入官方文档或你自己的文档仓库。
2. 检测文档存储库的版本（9、8、main/latest）并提供版本链接。
3. 通过 Web 界面，使用友好的图形用户界面(GUI)一键式代码检查(linting)和拼写检查。
4. 为本地部署提供自动启动功能。
5. 适用于带有 Web 管理控制台的 Rocky Linux 9 镜像，通过该控制台你可以配置所需的全部功能（计划中）。
6. 易于编写和更新的模块化方法。

## 架构

- 文档开发容器(localdocs-base) - 包含在你自己的文档分支上进行代码检查、拼写检查、构建和预览所需的所有工具。
- 基于 Web 的开发用户界面(WebDev UI) - 一个集成的开发环境(IDE)，用于文档的本地开发。
- 管理容器(admin) - 一个基于 Rocky Linux 9 的容器，带有 Web 用户界面，能够配置和管理本地文档开发环境。

容器镜像和 Web UI 是开源的。你可以通过以下地址获取：

- Rocky Docs WebDev - [GitHub 仓库](https://github.com/rocky-linux/rockydocs-webdev/)
- Localdocs-base 镜像 - [Docker Hub](https://hub.docker.com/r/rockylinux/localdocs-base)
- localdocs 镜像 - [Docker Hub](https://hub.docker.com/r/rockylinux/localdocs)

## 快速开始

系统要求：

- 基于RPM（红帽系列）的 Linux 发行版（Rocky Linux 8 或 9、CentOS Stream（8 或 9）、RHEL、AlmaLinux）。
- Git、`curl`、`python3-pip` 和 Podman。
- Firefox 或 Chromium 浏览器。

在某些情况下，你可能希望安装 Cockpit Web 控制台并为 Podman 添加插件。

```bash
sudo dnf install cockpit cockpit-podman
```

然后：

```bash
sudo systemctl enable --now cockpit.socket
```

启动一个终端，并运行以下脚本开始使用。在此过程中，你可能需要根据提示输入 root 密码。

```bash
curl -s https://raw.githubusercontent.com/rocky-linux/rockydocs-webdev/main/install.sh | bash
```

## 配置

在构建和启动容器之后，你需要进行配置。

按照终端中的操作指南完成其余配置，包括升级本地环境。该脚本会自动打开你的 Web 浏览器，打开地址为 `http://localhost:8081`

!!! note

    如果你安装在没有 Web 浏览器的无头(headless)服务器上，你可以使用另一台机器的浏览器，访问服务器的 IP 地址和端口 8081。你需要确保防火墙允许访问该端口。

在 Web 浏览器中，你应该能看到 WebDev UI：

![RockyDocs WebDev UI](../images/rockydocs_webdev_pointclick.png)

## 使用环境

### 首次运行

要在 VSCode 中编辑文档，请点击 VSCode "代码编辑器(Code Editor)"按钮。这将在你的 Web 浏览器中启动 VSCode。进入 `/home/localdocs/rockydocs/website/docs` 目录，即可找到你克隆的文档仓库。

### 运行 `mkdocs`

点击 "启动 Start" 按钮运行 `mkdocs`。

### 预览

在运行 `mkdocs` 后，任何目录的共享映射都会在浏览器 VSCode 编辑器中标识一个预览链接。点击该链接即可预览你的工作。

### 代码检查和拼写检查

以下截图重点展示了代码检查和拼写检查功能。点击 "编辑文档(Edit Docs)" 下的词法分析器(Lexer)或拼写检查器(Spell Checker)链接，即可运行相应的检查。

![Lexer and Spell Check](../images/rockydocs_webdev_lexer.png)

### 结束使用

当你处理完文档后，返回 WebDev UI 并点击 "停止(Stop)" 按钮来停止 `mkdocs`。

## 更新环境

当你在终端中更新本地环境时，需要执行以下命令：

```bash
curl -s https://raw.githubusercontent.com/rocky-linux/rockydocs-webdev/main/install.sh | bash
```

该脚本将识别当前有一个活跃的 localdocs 分支，并询问你是否希望从头开始执行安装。你无需这样做，该脚本会更新当前环境并退出。

你需要在当前终端标签页中重新加载环境，让脚本的更新生效。在 bash 中，执行：

```bash
source ~/.bashrc
```

你可以通过再次启动 install 命令来确认环境是否已更新。如果一切已是最新，你将看到以下提示：

```bash
Nothing to do. The installation is up to date
All operations completed
```

## 结论

`rockydocs-webdev` 部署是构建和查看文档本地副本的最佳方式。它能满足你目前大部分的需求，并且未来还有较大的扩展空间。使用它，你可能会爱上全新的文档处理体验！
