---
title: 使用 git pull 和 git fetch
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - Git
  - git pull
  - git fetch
---

## 简介

这篇 Gemstone 解释了 `git pull` 和 `git fetch` 命令之间的区别。它还说明了何时恰当使用每个命令。

## Git Fetch vs Git Pull

### Git Fetch

git fetch 从远程仓库下载更改，但不会将它们集成到您的本地分支中。

在未将更改合并到本地分支之前查看他人提交的内容，这很有用。

1. 列出当前检出的分支

     ```bash
     git branch
     ```

2. 从名为 origin 的远程仓库的主分支获取更改。输入：

     ```bash
     git fetch origin main
     ```

3. 比较本地仓库 HEAD 和远程 origin/main 仓库之间的更改。

     ```bash
     git log HEAD..origin/main
     ```

### Git Pull

Git Pull 下载更改并将其合并到您的当前分支。
它有助于快速用远程仓库的更改更新本地分支。

1. **拉取并合并更改**：

     ```bash
     git pull origin main
     ```

2. **查看合并的更改**：

     ```bash
     git log
     ```

## 附加说明

- **使用 `git fetch`**：
-- 当您需要在合并前查看更改时。
-- 避免本地分支中不必要的更改或冲突。

- **使用 `git pull`**：
-- 当您想用最新提交更新本地分支时。
-- 用于快速、直接的更新，无需先审查更改。

## 总结

理解 `git fetch` 和 `git pull` 之间的区别对于有效的 Git 工作流管理至关重要。在使用诸如 GitHub、GitLab、Gitea 等版本控制系统进行工作或协作时，根据您的需求选择正确的命令非常重要。
