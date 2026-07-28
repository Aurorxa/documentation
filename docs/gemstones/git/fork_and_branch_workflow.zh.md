---
title: Fork and Branch Git 工作流
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - GitHub
  - git
  - gh
  - git fetch
  - git add
  - git pull
  - git checkout
  - gh repo
---

## Fork and Branch 工作流

在此工作流类型中，贡献者将主编仓库 fork 到自己的 GitHub 账户，为他们的工作创建功能分支，然后通过 pull request 从这些分支提交贡献。

这篇 Gemstone 逐步介绍了如何设置本地仓库来为 GitHub 项目做贡献。它从最初的项目 forking 开始，设置本地和远程仓库，提交更改，并创建一个 pull request (PR) 来提交您的贡献。

## 前置条件

- 一个 GitHub 账户。
- 系统上已安装 `git` 和 `GitHub CLI (gh)`。
- 在 GitHub 上拥有项目的个人 fork。

## 操作步骤

1. 如果尚不存在，使用 gh 工具创建项目 fork。输入：

   ```bash
   gh repo fork rocky-linux/documentation --clone=true --remote=true
   ```

   此 *gh repo fork* 命令中使用的选项含义：
   - `--clone=true`：将 fork 的仓库克隆到您的本地机器。
   - `--remote=true`：将原始仓库添加为 remote，使您可以同步未来的更新。

2. 进入本地仓库目录。输入：

   ```bash
   cd documentation
   ```

3. 验证所有相关的远程仓库已在本地仓库中正确配置，输入：

   ```bash
   git remote -vv
   ```

4. 从上游 remote 获取最新更改：

   ```bash
   git fetch upstream
   ```

5. 创建并检出名为 your-feature-branch 的新功能分支：

   ```bash
   git checkout -b your-feature-branch
   ```

6. 进行更改，添加新文件，并将更改提交到本地仓库：

   ```bash
   git add .
   git commit -m "Your commit message"
   ```

7. 与名为 `upstream` 的远程仓库的主分支同步：

   ```bash
   git pull upstream main
   ```

8. 将更改推送到您的 Fork：

   ```bash
   git push origin your-feature-branch
   ```

9. 最后，使用 `gh` CLI 应用程序创建 Pull Request (PR)：

   ```bash
   gh pr create --base main --head your-feature-branch --title "Your PR Title" --body "Description of your changes"
   ```

   此 *gh pr create* 命令中使用的选项含义：

   `--base` main：指定上游仓库中将合并更改的目标分支。
   `--head` your-feature-branch：指示来自您 fork 的包含更改的源分支。
   `--title` "Your PR Title"：设置 pull request 的标题。
   `--body` "Description of your changes"：提供 pull request 中更改的详细描述。

## 总结

Fork and Branch 工作流是另一种常见的协作技术。
涉及的概要步骤如下：

1. Fork 仓库：在您的 GitHub 账户上创建项目仓库的个人副本。
2. 克隆 Fork：将您的 fork 克隆到本地机器以进行开发工作。
3. 设置上游 Remote：为了与主项目的变化保持同步，将原始项目仓库添加为 'upstream' remote。
4. 创建功能分支：为每个新功能或修复从更新的主分支创建一个新分支。分支名称应描述功能或修复。
5. 提交更改：进行更改并使用清晰简洁的提交信息提交。
6. 与上游同步：定期将您的 fork 和功能分支与上游主分支同步，以整合新更改并减少合并冲突。
7. 创建 Pull Request (PR)：将您的功能分支推送到 GitHub 上的 fork，并针对主项目打开 PR。您的 PR 应清晰描述更改并链接到任何相关 issue。
8. 响应反馈：在 PR 被合并或关闭之前协作处理审查反馈。

优势：

- 将开发工作隔离到特定分支，保持主分支的清洁。
- 使审查和集成更改更加容易。
- 减少与主项目不断演进的代码库发生冲突的风险。
