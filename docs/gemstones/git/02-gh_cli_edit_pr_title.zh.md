---
title: 通过 CLI 编辑或更改现有 Pull Request 的标题
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - GitHub
  - Pull Request
  - 文档
  - CLI
---

## 简介

这篇 Gemstone 介绍了如何使用 GitHub web 界面和 CLI 编辑或更改 GitHub 仓库中已有 pull request (PR) 的标题。

## 问题描述

有时，PR 创建后的标题可能需要修改以更好地反映当前的更改或讨论。

## 前置条件

- 一个已有的 GitHub pull request。
- 访问 GitHub web 界面或 CLI 并拥有必要的权限。

## 操作步骤

### 使用 GitHub CLI

1. **检出对应分支**：
   - 确保您在 PR 关联的分支上。

     ```bash
     git checkout branch-name
     ```

2. **使用 CLI 编辑 PR**：
   - 使用以下命令编辑 PR：

     ```bash
     gh pr edit PR_NUMBER --title "New PR Title"
     ```

   - 将 `PR_NUMBER` 替换为您的 pull request 编号，将 `"New PR Title"` 替换为所需的标题。

## 附加信息（可选）

- 编辑 PR 标题不会影响其讨论线程或代码更改。
- 如果对 PR 标题进行了重大更改，通知协作者是良好的实践。

## 总结

按照这些步骤，您可以通过 GitHub CLI 工具 (gh) 轻松更改 GitHub 仓库中已有 pull request 的标题。
