---
title: Git 中 Tracking 与 Non-Tracking 分支
author: Wale Soyinka
contributors:
tags:
  - git
  - git branch
  - Tracking Branch
  - Non-Tracking Branch
---

## 简介

这篇 Gemstone 深入探讨了 Git 中的 tracking（跟踪）分支和 non-tracking（非跟踪）分支。它还包含了验证和转换这些分支类型的步骤。

## Tracking Branch（跟踪分支）

跟踪分支是与远程分支相关联的分支。

1. 创建一个名为 my-local-branch 的新分支。使新本地分支跟踪名为 origin 的远程仓库的主分支。输入：

    ```bash
    git checkout -b my-local-branch origin/main
    ```

2. 使用 `git branch -vv` 命令验证该分支是跟踪分支。输入：

    ```bash
    git branch -vv
    ```

    查找带有 `[origin/main]` 的分支，表示它们正在跟踪 `origin/main`。

## Non-Tracking Branch（非跟踪分支）

非跟踪分支是独立于远程分支运行的分支。

1. 创建一个名为 my-feature-branch 的新非跟踪本地分支。输入：

    ```bash
    git checkout -b my-feature-branch
    ```

2. 非跟踪分支在 git branch -vv 输出中旁边不会显示远程分支。检查 my-feature-branch 是否为非跟踪分支。

## 将 Non-Tracking 转换为 Tracking

1. 可选地，首先确保主分支的最新更改已合并到目标分支。输入：

     ```bash
     git checkout my-feature-branch
     git merge main
     ```

2. 设置对远程分支的跟踪：

     ```bash
     git branch --set-upstream-to=origin/main my-feature-branch
     ```

     输出：`Branch 'my-feature-branch' set up to track remote branch 'main' from 'origin'.`

3. 验证更改。输入：

     ```bash
     git branch -vv
     ```

     现在，`my-feature-branch` 应该显示 `[origin/main]`，表示它正在跟踪。

## 总结

理解跟踪和非跟踪分支之间的细微差别在 Git 中至关重要。这篇 Gemstone 阐明了这些概念，并演示了如何识别和转换这些分支类型，以实现最佳的 git 工作流管理。
