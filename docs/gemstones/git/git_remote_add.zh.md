---
title: 使用 git CLI 添加远程仓库
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - GitHub
  - git
  - git remote
  - git fetch
---

## 简介

这篇 Gemstone 说明了如何使用 Git 命令行界面向已有的 FOSS 项目本地克隆添加特定的远程仓库。
我们将使用 Rocky Linux 文档项目的仓库作为 FOSS 项目示例 - <https://github.com/rocky-linux/documentation.git>

## 前置条件

- 一个 GitHub 账户。
- 系统上已安装 `git`。
- FOSS 项目仓库的本地克隆。

## 操作步骤

1. 打开终端并将工作目录切换到包含项目本地克隆的文件夹。
   例如，如果您将 github 仓库克隆到 ~/path/to/your/rl-documentation-clone，输入

     ```bash
     cd ~/path/to/your/rl-documentation-clone
     ```

2. 在进行任何更改之前，列出已配置的 remote。输入：

   ```bash
   git remote -vv
   ```

   如果这是一个刚克隆的仓库，您可能会在输出中看到一个名为 `origin` 的单独 remote。

3. 将 Rocky Linux 文档仓库（`https://github.com/rocky-linux/documentation.git`）作为新 remote 添加到您的本地仓库。此处我们将 upstream 指定为此 remote 的名称。输入：

     ```bash
     git remote add upstream https://github.com/rocky-linux/documentation.git
     ```

4. 为了进一步强调分配给远程仓库的名称是任意的，创建一个名为 rocky-docs 的另一个 remote，指向同一个仓库，运行：

   ```bash
   git remote add rocky-docs https://github.com/rocky-linux/documentation.git
   ```

5. 确认新的远程仓库已成功添加：

     ```bash
     git remote -v
     ```

     您应该看到 `upstream` 及其 URL 已列出。

6. 可选地，在开始对本地仓库进行任何更改之前，您可以从新添加的 Remote 获取数据。
   通过运行以下命令从新添加的 remote 获取分支和提交：

     ```bash
     git fetch upstream
     ```

## 附加说明

- *Origin*：这是 Git 给克隆来源的远程仓库分配的默认名称。它类似于仓库 URL 的昵称。当您克隆一个仓库时，该远程仓库会自动在本地 Git 配置中设置为 "origin"。这个名称是约定俗成的，但并非强制。

- *Upstream*：这通常指您 fork 项目时的原始仓库。
   在开源项目中，如果您 fork 一个仓库进行更改，fork 的仓库是您的 "origin"，原始仓库通常被称为 "upstream"。这个名称是约定俗成的，但并非强制。

   origin 和 upstream 的使用/分配之间的微妙区别对于通过 pull request 为原始项目做贡献至关重要。

## 总结

git CLI 工具可以轻松地使用描述性名称将特定的远程仓库添加到 FOSS 项目的本地克隆中。这使得您能够有效地与各种仓库同步并做出贡献。
