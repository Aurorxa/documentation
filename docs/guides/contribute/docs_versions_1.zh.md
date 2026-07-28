---
title: 使用双远程(Double Remote)进行文档版本管理
author: Steven Spencer
contributors: Ganna Zhyrnova
tags:
  - contributing
  - documentation
  - versioning
---

## 引言

2025年初秋，文档团队从覆盖所有版本的单一文档版本转变为每个版本拥有自己的文档分支。这使得区分不同版本之间的说明变得更加容易。然而，它**确实**使编写或修订文档的过程复杂化了，尤其是针对较旧版本（Rocky Linux 8 或 9）的文档。本文档概述了一种使用双远程方法处理此流程的策略。

!!! info "Rocky Linux 版本"

    截至本文日期，即 2025 年 10 月，版本情况如下：

    | Branch | Version |
    |--------|---------|
    | main   | Rocky Linux 10 |
    | rocky-9 | Rocky Linux 9 |
    | rocky-8 | Rocky Linux 8 |

## 前提条件

* 一个个人 GitHub 账号，已[配置 SSH 密钥](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
* 已有的 Rocky Linux 文档 fork
* 对命令行中使用 `git` 有一定了解，或愿意学习
* 已安装 `git` 工具

## 克隆仓库(Repository)

克隆 Rocky Linux 仓库会将 Rocky Linux 文档的副本移动到您工作站的 `/documentation` 目录中。您可能曾在某些地方或其他 GitHub 项目中看到过，应该始终从您个人 fork 的项目进行克隆。但在这种情况下，为了使克隆能感知版本，情况并非如此。您需要从 Rocky Linux 项目进行克隆。本文档将随着进展解释为什么这样做。此外，您需要重命名 git 远程(remote)名称，使其在逻辑上合理（Rocky Linux 为 "upstream"，您的 GitHub 为 "origin"）。

1. 克隆 Rocky Linux 文档：

    ```bash
    git clone git@github.com:rocky-linux/documentation.git
    ```

1. 进入 `/documentation` 目录：

    ```bash
    cd documentation
    ```

1. 检查远程名称：

    ```bash
    git remote -v
    ```

    这将显示：

    ```bash
    origin git@github.com:rocky-linux/documentation.git (fetch)
    origin git@github.com:rocky-linux/documentation.git (push)
    ```

    您希望此资源为 "upstream" 而非 "origin"。

1. 更改远程名称

    ```bash
    git remote rename origin upstream
    ```

    再次运行 `git remote -v` 现在将显示：

    ```bash
    upstream git@github.com:rocky-linux/documentation.git (fetch)
    upstream git@github.com:rocky-linux/documentation.git (push)

    ```

## 将您的 fork 添加为远程

在添加并正确命名 Rocky Linux 远程后，您需要将个人 GitHub fork 设置为 origin 远程。

1. 在此步骤中，您需要知道您的 GitHub 用户名，您应该已经知道。将 "[username]" 字段替换为正确的名称。添加您的远程：

    ```bash
    git remote add origin git@github.com:[username]/documentation.git
    ```

1. 检查您的 git 远程：

    ```bash
    git remote -v
    ```

    这将显示：

    ```bash
    origin git@github.com:[username]/documentation.git (fetch)
    origin git@github.com:[username]/documentation.git (push)
    upstream git@github.com:rocky-linux/documentation.git (fetch)
    upstream git@github.com:rocky-linux/documentation.git (push)
    ```

## 检查更新并将版本分支添加到您的 fork

1. 添加远程后，首先从 upstream 拉取更新并推送到 origin。如果您刚刚创建了 fork 和远程，那么将没有更新需要推送，但以以下步骤开始是个好习惯：

    ```bash
    git pull upstream main && git push origin main
    ```

1. 使您的本地克隆感知 `upstream` 上存在的分支：

    ```bash
    git fetch upstream
    ```

1. 检出两个较旧版本分支之一：

    ```bash
    git checkout rocky-8
    ```

    !!! warning "如果您的克隆来自您的 fork，这将不起作用。"

        这就是为什么克隆过程是从 Rocky Linux 进行，而不是从您的 fork 进行。您的 fork 不会感知到旧的分支。要获得以下消息，您**必须**从 Rocky Linux 克隆本地文档仓库。

    如果正确设置了远程，您现在应该看到：

    ```bash
    branch 'rocky-8' set up to track 'upstream/rocky-8'.
    Switched to a new branch 'rocky-8'
    ```

    这有效地创建了一个本地分支 `rocky-8`。下一步是从 'rocky-8' 拉取任何更改并推送到您的 origin。本地应该没有任何更改，但该分支在您的 fork 上不存在，因此此过程将创建它：

    ```bash
    git pull upstream rocky-8 && git push origin rocky-8
    ```

    您可能会收到一条消息，提示您可以从推送创建 pull request。可以忽略此消息。实际发生的是您的 fork 现在有了一个 `rocky-8` 分支。

1. 检出另一个旧分支 (`rocky-9`) 并重复刚才对该分支执行的步骤。

完成后，您的本地 fork 和克隆上现在将有 `main`、`rocky-8` 和 `rocky-9` 分支，并且能够在这些分支中的任何一个上编写文档。

## 在旧版本上编写或更新文档

如果您熟悉针对文档 `main` 分支编写 pull request (PR)，此过程仍然一如既往地有效。只需记住 `main` 适用于最新版本（本文编写时为版本 10）。要对较旧版本进行小更改，首先需要基于该分支创建一个本地编辑分支。为此，请将 `-b` 选项与 `git checkout` 命令一起使用。此命令创建一个名为 `8_rkhunter_changes` 的分支，并将其基于 `rocky-8` 分支：

```bash
git checkout -b 8_rkhunter_changes rocky-8
```

您现在可以编辑要更改的文件，它将使用来自 `rocky-8` 分支的该文档版本。

编辑完成后，照常保存、暂存(stage)和提交(commit)您的更改，然后将更改推送到 `origin` 远程：

```bash
git push origin 8_rkhunter_changes
```

但是，在创建 PR 时，GitHub 会自动将其视为对 `main` 分支的更改，即使您在修改文档时专门使用了 `rocky-8` 分支。在看到此错误的比较页面时，请不要过快地创建 PR：

![Wrong comparison](../images/incorrect_comparison_branchb_blur.png)

这里需要做的是将比较分支更改为正确的（本例中为 `rocky-8`）分支：

![Right comparison](../images/correct_comparison_branch_blur.png)

纠正比较分支后，继续创建 PR 并等待其被合并。

## 合并后更新您的旧版本分支

与 `main` 分支一样，保持旧版本分支与所有更改同步是个好主意。以下命令集将更新**所有**版本，使其与 upstream 匹配：

```bash
git checkout rocky-8
git pull upstream rocky-8 && git push origin rocky-8
git checkout rocky-9
git pull upstream rocky-9 && git push origin rocky-9
git checkout main
git pull upstream main && git push origin main
```

完成这些命令后，所有本地分支和 fork 都将是最新的。

## 结论

本文档引导您了解了一种双远程策略，用于处理自文档版本创建以来的新文档或修订。
