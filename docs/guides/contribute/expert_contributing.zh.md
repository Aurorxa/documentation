---
title: 专家贡献指南
authors: 多位作者
contributors: Ganna Zhyrnova
---

# 专家贡献指南

Rocky Linux 文档项目欢迎并感谢社区所有人的贡献。有时，新的贡献者可能会对如何开始感到困惑，需要一些指导。本指南以及"首次贡献者"指南旨在帮助您。请注意，本指南适用于中级到高级用户，而[首次贡献者指南](https://github.com/rocky-linux/documentation/blob/main/README.md)面向初学者。

## 贡献前

与 Rocky Linux 的所有其他方面一样，文档团队始终以文档化 Rocky Linux 工作流为优先。文档**必须**与 Rocky Linux 的稳定性、更新政策以及开源精神保持一致。换句话说，文档需要与 Rocky Linux 相关。

如果您不确定您的文档是否符合，请随时提出！团队中有一个非常活跃的[文档 Mattermost 频道](https://chat.rockylinux.org/rocky-linux/channels/documentation)，也欢迎您在 [GitHub Issues](https://github.com/rocky-linux/documentation/issues) 中提出新问题。以 "Documentation Idea:" 开头创建问题。这些都可以极大地帮助确保您的文档符合 Rocky Linux 的目标和理念。

## Rocky Linux 全栈文档

当 Rocky Linux 首次发布时，有两组独立的文档：Rocky Linux 安装文档和 文档网站上的文档。本文档涵盖了文档部分。目前有两个 GitHub 仓库构成了[文档部分](https://docs.rockylinux.org)的所有内容。这些仓库是：

* [文档仓库](https://github.com/rocky-linux/documentation)
* [文档站点仓库](https://github.com/rocky-linux/docs.rockylinux.org)

您只需要关注第一个仓库，用于贡献文档。第二个仓库是文档站点自身代码的存储库。只有在您打算为文档站点的构建方式（或者可能是其外观）做出贡献时，才需要关注它。

## 贡献所需技能

### `git` 的基本技能

GitHub 仓库使用 `git` 进行版本控制。如果您不是经验丰富的 `git` 用户，那没关系。您可以在本地工作站上遵循[此流程](localdocs/local_docs.md)操作，它适合那些对 `git` 不太熟悉的人，并编写您的内容。准备好后，联系团队，他们可以帮助您完成内容，使用 `git`。如果您想深入学习 `git`，互联网上有一些很棒的资源：

<!-- markdownlint-disable-next-line MD036 -->

* [What is Git?](https://git-scm.com/) —— 由 Atlassian 撰写的出色的 Git 指南。
* [Git 书籍](https://git-scm.com/book/en/v2) —— 官方 Git 书籍。

### 格式化文档

#### Markdown

如果您对 `git` 和 Markdown 完全不熟悉，可能需要一些时间才能让所有内容都准确无误。但不要因此而气馁！如果您有任何疑问、需要帮助或希望对文档进行审查，请随时联系文档团队。他们随时为您提供帮助！

Markdown 针对所有文档进行了标准化，因此请确保阅读我们的[格式化指南](rockydocs_formatting.md)。任何贡献文档的人还必须熟悉 `git` 的使用。如果您愿意花一些时间学习 Markdown，学习 `git` 只是这些原则的自然延伸。学习和使用 `git` 有很多好处，其中之一就是您的本地修改可以轻松地在 `mkdocs` 中呈现，这将为您展示您的文档在网上呈现时的预览效果。

<!-- markdownlint-disable-next-line MD036 -->

* [Markdown 指南](https://www.markdownguide.org/) —— 一本全面的 Markdown 语法指南。
* [另一本 Markdown 指南](https://guides.github.com/features/mastering-markdown/#what) —— GitHub 的 Markdown 快速指南。

#### 文档格式化指南

如前所述，文档团队已经制定了一套关于格式化文档以及如何提交文档进行审批的常规标准。请务必阅读这些指南，以简化您的文档提交流程：

* [文档格式化指南](rockydocs_formatting.md)
* [首次贡献者指南](https://github.com/rocky-linux/documentation/blob/main/README.md)
* [关于版本化文档的指南](docs_versions_1.md)

## 翻译文档

我们还在持续调整翻译工作流，以使所有相关方的生活尽可能简单。尽管存在一些痛点，但欢迎您参与贡献，并且考虑到我们最初的翻译人力有限，我们感谢任何能够帮助将文档翻译成本地语言的人。

在撰写本文时，以下是翻译大量文档的建议工作流：

1. 前往 [Rocky Linux Weblate](https://translate.rockylinux.org/projects/documentation/)，该页面将列出所有可用的翻译语言。每种语言都会包含一个需要翻译的文档列表。
2. 打开某种语言，您将看到一个页面，左侧是原始文本，右侧显示您语言的翻译状态。根据翻译进度，您可能会看到绿色和红色进度条。
3. 选择一个组件继续翻译。
4. Weblate 界面提供了非常方便的操作方式和有用的帮助。
5. 您的翻译完成后，将被提交回 GitHub repo 进行同步。
6. 同步完成后，文档管理员会将翻译合并到文档中。

请注意，您需要一个 GitHub 账号并登录 Weblate 来保存您的翻译。

如果您对 Weblate 有任何疑问，请查阅[文档](https://docs.weblate.org/en/latest/index.html)，或联系文档团队。

## Python 虚拟环境(Virtual Environment)设置

如果您熟悉 `git` 并希望尝试本地副本，可以使用此方法，它也可以在[此处](localdocs/mkdocs_venv.md)找到。

### Python VENV 方法 —— 前提条件和假设

运行此环境所需的前提条件：

* 安装了 Python3，根据您的操作系统版本可能需要使用 `dnf install python3`。
* 安装了 `pip3`，根据您的操作系统版本可能需要使用 `dnf install python3-pip`。
* Git，使用`dnf install git`。
* [使用公钥的 SSH 设置](../security/ssh_public_private_keys.md)
* Python `venv` 模块（`python3-venv` 软件包）。根据您的操作系统可能需要使用 `dnf install python3-venv`。

### 首次设置（假设您已按先决条件中的描述安装了需求）

以下步骤假设您要从头开始执行过程。

* 克隆 docs.rockylinux.org 仓库：

    ```bash
    git clone git@github.com:rocky-linux/docs.rockylinux.org.git
    ```

* 完成后，进入 docs.rockylinux.org 目录：

    ```bash
    cd docs.rockylinux.org
    ```

* 确保您在存储库中当前位于正确的分支。目前仓库使用 `main` 分支。运行以下命令查看您在仓库中的位置：

    ```bash
    git branch
    ```

    输出应显示当前分支带有星号，如下所示：

    ```bash
    * main  
    ```

    在现代 Git 版本中，输出可能显示以下内容：

    ```bash
    * (main)
    ```

* 现在创建 Python VENV（您会注意到，在您安装的 Python 版本中首次使用时，这不仅会创建 VENV，还会更新 `pip`）：

    ```bash
    python3 -m venv venv
    ```

* 激活 VENV：

    ```bash
    source venv/bin/activate
    ```

    您会注意到提示符发生了变化，显示您正在 VENV（虚拟环境）中：

    ```bash
    (venv) [rocky_pc docs.rockylinux.org]$
    ```

* 接下来，安装 `requirements.txt` 文件中的要求：

    ```bash
    pip install -r requirements.txt
    ```

    这将成功安装所有必要模块。

* 现在，使用以下命令克隆 Rocky Linux 文档仓库：

    ```bash
    git clone git@github.com:rocky-linux/documentation.git docs
    ```

    现在尝试构建和提供文档：

    ```bash
    mkdocs serve
    ```

* 假设一切正常，您将看到一则免责声明，说明不打算将其用作生产网络服务器，还有大量调试信息，最后是一行通知，简单说明此实例在 localhost:8000 上提供服务。要查看文档，您需要打开 Web 浏览器并输入以下 URL：

  ```text
  http://localhost:8000
  ```

  如果一切正常，您应该看到文档站点的本地实例。

  - 注意：完成后，您可以简单地删除 docs.rockylinux.org 目录及其所有内容，所有这些就会消失。您的计算机上不会留下任何软件包或垃圾残留。

## 从 GitHub Web 界面贡献

您可以从 GitHub Web 界面完成所有文档。这有时比使用上述方法更简单。**但是**，您还必须知道如何进行 pull request (PR)。

### 创建 PR

您可以对主要仓库进行 PR。这意味着将您的更改从您的副本推送到主要仓库。我们有一个可点击的指南，解释了执行此操作时的屏幕截图。您可以在[此处](../assets/images/create-pull-request.gif)找到它。

### PR 技巧

1. 在文档方面，请尽量保持简单。即一次一个 pull request (PR)。
2. 确保创建 PR 后，文档工作流中的检查全部通过。如果它们不确定，请要求澄清。
3. 不要创建一个 PR 然后弃之不管。在更新后与维护人员进行对话。您投入了很多工作，维护人员希望确保它被合并。
4. 当您开始对文档仓库进行 PR 时，会通过电子邮件向您发送任何标记。这些邮件极其重要，请务必阅读！如果审查过程没有标记，检查可能会毫无理由地失败，因为需要标记才能允许工作流运行。
5. **请务必阅读** [首次贡献者指南](https://github.com/rocky-linux/documentation/blob/main/README.md)。
6. 如果您没有从审查人员那里听到任何消息，请随时在 PR 中添加评论，或将其标记给团队中的某人。耐心是美德，但有时人们会忙，特别是如果这是一个志愿者项目。不要让这些使您气馁。我们想要您的 PR！联系我们！

## 结论

我们希望这是一份全面的指南，涵盖了向 Rocky Linux 文档贡献的所有主要方面。如果存在问题或需要更多信息，欢迎提出。
