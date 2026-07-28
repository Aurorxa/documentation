---
title: 通过 CLI 首次向 Rocky Linux 文档贡献
author: Wale Soyinka
contributors:
tags:
  - GitHub
  - Rocky Linux
  - 贡献
  - Pull Request
  - CLI
---

## 简介

这篇 Gemstone 详细介绍了如何仅使用命令行界面 (CLI) 为 Rocky Linux 文档项目做出贡献。它涵盖了首次 fork 仓库并创建 pull request。
我们以贡献一篇新的 Gemstone 文档为例。

## 问题描述

贡献者可能更倾向于或需要用 CLI 执行所有操作，从 fork 仓库到首次提交 pull request。

## 前置条件

- 一个 GitHub 账户
- 系统上已安装 `git` 和 `GitHub CLI (gh)`
- 准备好一个待贡献的 markdown 文件

## 解决方案步骤

1. **使用 GitHub CLI Fork 仓库**：
   Fork 上游仓库到您的账户。

   ```bash
   gh repo fork https://github.com/rocky-linux/documentation --clone
   ```

2. **进入仓库目录**：

   ```bash
   cd documentation
   ```

3. **添加上游仓库**：

   ```bash
   git remote add upstream https://github.com/rocky-linux/documentation.git
   ```

4. **创建新分支**：
   为您的贡献创建一个新分支。输入：

   ```bash
   git checkout -b new-gemstone
   ```

5. **添加您的新文档**：
   使用您喜欢的文本编辑器创建和编辑新的贡献文件。
   对于本示例，我们将创建一个名为 `gemstome_new_pr.md` 的新文件，并将其保存在 `docs/gemstones/` 目录下。

6. **提交您的更改**：
   暂存并提交您的新文件。输入：

   ```bash
   git add docs/gemstones/gemstome_new_pr.md
   git commit -m "Add new Gemstone document"
   ```

7. **推送到您的 Fork**：
   将更改推送到您 fork 的 Rocky Linux 文档仓库副本。输入：

   ```bash
   git push origin new-gemstone
   ```

8. **创建 Pull Request**：
   向上游仓库创建一个 pull request。

   ```bash
   gh pr create --base main --head wsoyinka:new-gemstone --title "New Gemstone: Creating PRs via CLI" --body "Guide on how to contribute to documentation using CLI"
   ```

## 附加信息（可选）

- 使用 `gh pr list` 和 `gh pr status` 跟踪您的 pull request 状态。
- 查看并遵循 Rocky Linux 文档项目的贡献指南。

## 总结

按照这些步骤，您应该能够完全通过 CLI 成功创建您的第一个 PR 并向 Rocky Linux 文档仓库做出贡献！
