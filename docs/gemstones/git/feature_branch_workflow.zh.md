---
title: Git 中的功能分支工作流（Feature Branch Workflow）
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - git
  - Feature Branch Workflow
  - GitHub
  - gh
  - git fetch
  - git add
  - git pull
  - git checkout
---

## Feature Branch Workflow（功能分支工作流）

这种流行的 git 工作流涉及在主编仓库中直接为每个新功能或修复创建新分支。
它通常用于贡献者对仓库有直接推送权限的项目中。

这篇 Gemstone 概述了使用 Git Feature Branch Workflow 为 `rocky-linux/documentation` 项目设置本地仓库以进行工作和贡献的过程。

用户 "rockstar" 已 fork 了此仓库，我们将使用 `https://github.com/rockstar/documentation` 作为 origin。

## 前置条件

- 一个 GitHub 账户和一个项目的 fork（例如，`https://github.com/rockstar/documentation`）。
- 已安装 `git` 和 `GitHub CLI (gh)`。

## 操作步骤

1. 如果尚未完成，克隆您的 fork：

   ```bash
   git clone https://github.com/rockstar/documentation.git
   cd documentation
   ```

2. 添加上游 remote：

   ```bash
   git remote add upstream https://github.com/rocky-linux/documentation.git
   ```

3. 获取上游更改：

   ```bash
   git fetch upstream
   ```

4. 创建新的功能分支：

   ```bash
   git checkout -b feature-branch-name
   ```

5. 进行更改，添加新文件并提交：

     ```bash
     git add .
     git commit -m "Implementing feature X"
     ```

6. 保持分支更新。定期合并上游更改以避免冲突：

     ```bash
     git pull upstream main --rebase
     ```

7. 推送到您的 fork，输入：

   ```bash
   git push origin feature-branch-name
   ```

8. 创建 Pull Request：

   ```bash
   gh pr create --base main --head rockstar:feature-branch-name --title "New Feature X" --body "Long Description of the feature"
   ```

## 总结

Feature Branch 工作流是一种常见的协作技术，允许团队同时在项目的各个方面进行工作，同时保持稳定的主代码库。

涉及的概要步骤如下：

1. 克隆主编仓库：直接将主项目仓库克隆到本地机器。
2. 创建功能分支：对于每个新任务，从主分支创建一个具有描述性名称的新分支。
3. 提交更改：在您的分支中处理功能或修复并提交更改。
4. 保持分支更新：定期从主分支合并或变基（rebase）以跟进最新更改。
5. 打开 Pull Request：功能完成后将分支推送到主编仓库，并打开一个 PR 供审查。
6. 代码审查和合并：经过审查和批准后，分支将被合并到主分支。

*优势*：

- 简化了具有直接仓库访问权限的常规贡献者的贡献流程。
- 确保每个功能在集成到主代码库之前都经过审查。
- 有助于保持干净和线性的项目历史。
