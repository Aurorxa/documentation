---
title: 本地文档介绍
author: Steven Spencer
contributors: Ganna Zhyrnova
tags:
  - local docs
  - docs as code 
  - linters
---

# 引言

使用 Rocky Linux 文档的本地副本对于经常贡献并需要确切了解文档在合并后网页界面中效果的人来说非常有用。这里包含的方法代表了截至目前贡献者的偏好。

使用文档的本地副本是那些信奉"代码即文档(docs as code)"理念的人们在开发流程中的一个步骤，这是一种类似于代码开发的文档编写工作流。虽然在本地文档部分中未包含，但[专家贡献指南](../expert_contributing.md)也适用于本地文档指南的范畴，同时包含了许多所有贡献者可能感兴趣的其他参考资料。您可以结合这里的其他文档查阅该文档，可能会发现它很有用。

## Markdown 代码检查(linting)

除了存储和构建文档的环境之外，一些作者可能还会考虑使用 markdown 的代码检查工具(linter)。Markdown 代码检查工具在编写文档的许多方面都很有帮助，包括语法、拼写、格式等检查。有时这些是独立的工具或编辑器插件。其中一个工具是 [markdownlint](https://github.com/DavidAnson/markdownlint)，一个 Node.js 工具。`markdownlint` 是许多流行编辑器（包括 Visual Studio Code 和 NVChad）的插件。因此，文档目录的根目录中包含一个 `.markdownlint.yml` 文件，该文件将应用项目中可用和启用的规则。`markdownlint` 完全是格式检查工具。它将检查多余的空格、内联 HTML 元素、双重空行、不正确的制表符等。对于语法、拼写、包容性语言使用等，请安装其他工具。

!!! info "免责声明"

    本类别（"本地文档"）中的所有项目都不是编写文档并提交审批所必需的。它们是为那些希望遵循[代码即文档(docs as code)](https://www.writethedocs.org/guide/docs-as-code/)理念的人而存在的，这些理念至少包括一份文档的本地副本。
