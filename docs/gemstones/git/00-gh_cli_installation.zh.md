---
title: 在 Rocky Linux 上安装和设置 GitHub CLI
author: Wale Soyinka
Contributor: Ganna Zhyrnova
tags:
  - GitHub CLI
  - gh
  - git
  - github
---

## 简介

这篇 gemstone 介绍了 GitHub CLI 工具 (gh) 在 Rocky Linux 系统上的安装和基本设置。此工具使用户能够直接从命令行与 GitHub 仓库进行交互。

## 问题描述

用户需要一种便捷的方式在不离开命令行环境的情况下与 GitHub 进行交互。

## 前置条件

- 运行 Rocky Linux 的系统
- 可以访问终端
- 基本熟悉命令行操作
- 拥有一个现有的 Github 账户

## 操作步骤

1. **使用 curl 安装 GitHub CLI 仓库**：
   使用 curl 命令下载 `gh` 的官方仓库文件。下载的文件将保存在 /etc/yum.repos.d/ 目录下。下载后，使用 dnf 命令从该仓库安装 `gh`。输入：

   ```bash
   curl -fsSL https://cli.github.com/packages/rpm/gh-cli.repo | sudo tee /etc/yum.repos.d/github-cli.repo
   sudo dnf -y install gh
   ```

2. **验证安装**：
   确保 `gh` 已正确安装。输入：

   ```bash
   gh --version
   ```

3. **对 GitHub 进行身份验证**：
   登录到您的 GitHub 账户。输入：

   ```bash
   gh auth login
   ```

   按照提示完成身份验证。

## 总结

现在您应该已经在 Rocky Linux 9.3 系统上安装并设置好了 GitHub CLI，可以直接从终端与 GitHub 仓库进行交互。

## 附加信息（可选）

- GitHub CLI 提供了各种命令，例如 `gh repo clone`、`gh pr create`、`gh issue list` 等。
- 更多详细用法，请参考 [官方 GitHub CLI 文档](https://cli.github.com/manual/)。
