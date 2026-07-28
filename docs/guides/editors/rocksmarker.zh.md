---
title: Rocksmarker —— Neovim IDE
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.6, 9.0
tags:
  - 编辑器
  - neovim
  - nvchad
  - rocksmarker
---

# Rocksmarker —— Neovim 集成开发环境(IDE)

## 引言

!!! warning "弃用声明"

    此安装脚本已被弃用，不再维护。

[Rocksmarker](https://github.com/ambaradan/rocksmarker) 是一个安装脚本，旨在提供一个完整的 NvChad 集成开发环境，专门为 Rocky Linux 文档贡献者量身定制。它会安装一个 NeoVim 编辑器，该编辑器预配置了许多有用的功能，旨在帮助文档贡献者更快、更准确地完成工作。

脚本包括：

* 支持 Rocky Linux 8 和 Rocky Linux 9
* 从源代码安装 NeoVim
* 一个由 [NvChad](https://nvchad.com/) 启动配置预先配置的环境，专门针对文档贡献
* [markdownlint-cli2](https://github.com/DavidAnson/markdownlint-cli2)、`vale`、`markdown-link-check`、`talosctl`、`gitleaks`、`pre-commit` 和 `cspell` 的安装和配置
* [Marksman LSP](https://github.com/artempyanykh/marksman) 用于对文档页面的实时诊断
* 代码检查(linting)、拼写检查和预提交(Pre-commit)的完整配置

## 移植到新环境

!!! warning

    由于运行 `rocksmarker` 会导致现有 Neovim 环境被新配置覆盖，因此**不推荐**在已有 Neovim 安装的环境中运行它。

自本文档最初编写以来，整个配置已被剥离出来，并放入一个配置仓库中，该仓库简化了安装，且不会影响可能已在运行的现有配置。

该配置现在名为 `RockyDocs Neovim`，其仓库位于 [此处](https://github.com/ambaradan/rockydocs-neovim)。未来的安装将基于该仓库完成。有关更多信息，请参阅该仓库的 README 文件。

## 结论

Rocksmarker IDE（现已弃用）为 Rocky Linux 文档贡献者提供了一个功能齐全的 NvChad 环境。新的 `rockydocs-neovim` 配置取代了它，提供更灵活、更安全的安装方式，不会覆盖现有的 Neovim 配置。
