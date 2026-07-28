---
title: 简介
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with:  9.4
tags:
  - lxd
  - incus
  - enterprise
---

# 构建 Incus 服务器

## Incus 在 Rocky Linux 上的状态

大约一年前，lxc-users 邮件列表上发布了以下公告：

> Canonical，LXD 项目的创建者和主要贡献者，已决定在经历 8 年多作为 Linux Containers 社区的一部分之后，该项目现在更应该直接归属 Canonical 自身的项目体系。

其中一个决定因素是 LXD 的一些主要开发者的辞职。这些开发者随后将 LXD 分支（fork）为 Incus，并于 2023 年 8 月宣布了该分支。一个发布版本（0.1）于 2023 年 10 月发布，开发者们随后通过逐步发布（从 0.7 到 2024 年 3 月）在该版本上快速构建。长期支持版本（6.0 LTS）在 0.7 版本之后发布。当前版本为 6.5。

在整个过程中，曾认为 Canonical 会继续维护与 Linux Containers 提供的容器镜像的链接。然而，[许可证变更](https://stgraber.org/2023/12/12/lxd-now-re-licensed-and-under-a-cla/)使得 Linux Containers 无法继续在 LXD 中提供容器镜像。这意味着 LXD 仍会有容器镜像，但可能与你期望的不同。如果你使用 Incus，Linux Containers 将继续托管和支持其镜像。

本文档是将 [LXD 手册](../lxd_server/00-toc.md) 转换为 Incus。由于 Rocky Linux 项目基础设施联合负责人 [Neil Hanlon](https://wiki.rockylinux.org/team/infrastructure/) 创建了仓库，你可以在 Incus 被纳入 EPEL（Extra Packages for Enterprise Linux，企业级 Linux 额外软件包仓库）之前安装它。

!!! warning "Incus 服务器在 Rocky Linux 8 上不可用"

    Incus 服务器安装仅适用于 Rocky Linux 9.x，目前已在 Rocky Linux 9.4 上测试。如果你需要能在 Rocky Linux 8.x 上运行的方案，请使用[前述的 LXD 步骤](../lxd_server/00-toc.md)。

## 简介

Incus 在[官方网站](https://linuxcontainers.org/incus/)上有最好的描述，但你也可以将其视为一个容器系统，它在容器中提供了虚拟服务器的优势。

它非常强大，并且通过适当的硬件和设置，可以在单台硬件上创建许多服务器实例。如果你将其与快照服务器配合使用，你将拥有一组在主服务器宕机时可以几乎立即启动的容器。

!!! warning "这不是备份"

    你不应将其视为传统的备份。你仍然需要常规的备份系统，例如 [rsnapshot](../../guides/backup/rsnapshot_backup.md)。

Incus 的学习曲线可能较陡，但本手册将尝试为你提供在 Rocky Linux 上部署和使用 Incus 的知识。

对于那些希望在笔记本电脑或工作站上将 Incus 用作实验环境的人，请参阅[附录 A：工作站设置](30-appendix_a.md)。

## 前提条件与假设

* 一台配置良好的 Rocky Linux 9 服务器。在生产环境中，考虑为 ZFS 磁盘空间单独使用一块硬盘（如果使用 ZFS 则必须如此）。是的，这里的假设是一台裸金属服务器（bare metal server），而不是 VPS（Virtual Private Server，虚拟专用服务器）。
* 这是一个高级主题，但并不难理解。如果你从一开始就按照这些说明操作，应该能够成功。话虽如此，了解一些关于容器管理的基础知识将会大有裨益。
* 在命令行操作方面得心应手，并熟练使用命令行编辑器。（这些示例中均使用 _vi_，但你可以替换为自己喜欢的编辑器。）
* 在大多数过程中，你需要使用非特权用户。在早期的设置步骤中，你需要是 root 用户，或者能够通过 `sudo` 成为 root 用户。在这些章节中，我们假设你的非特权用户为 "incusadmin"。你将在后续过程中创建此用户帐户。
* 对于 ZFS，确保 UEFI 安全启动（secure boot）**未**启用。否则，你必须对 ZFS 模块进行签名才能加载它。
* 主要使用基于 Rocky Linux 的容器

!!! info

    作者提供了一种使用 ZFS 文件系统的方法。请注意，Incus 项目推荐将 BTRFS 作为 Incus 服务器的文件系统。（ZFS 仍然是一个文件系统选项。）然而，BTRFS 在 Rocky Linux 9.4 上完全不可用。除非上游认证并在那里发布，否则你唯一的选择是使用 ZFS 或其他包含的文件系统。要了解更多关于 Incus 文件系统选项的信息，请参阅[该项目的官方文档。](https://linuxcontainers.org/incus/docs/main/reference/storage_dir/)

## 内容概要

* **第 1 章：安装与配置** 介绍了主服务器的安装。通常，在生产环境中正确使用 Incus 的方式是拥有一台主服务器和一台快照服务器。
* **第 2 章：ZFS 设置** 介绍了 ZFS 的设置和配置。ZFS 是一个由 Sun Microsystems 最初为其 Solaris 操作系统创建的开源逻辑卷管理器和文件系统。
* **第 3 章：Incus 初始化与用户设置** 介绍了基础的初始化与选项，以及你将在此后大部分过程中使用的非特权用户的设置。
* **第 4 章：防火墙设置** 包含 `firewalld` 设置选项。
* **第 5 章：镜像的设置与管理** 描述了将操作系统镜像安装到容器并进行配置的过程。
* **第 6 章：配置文件** 介绍了添加配置文件并将其应用于容器，主要涵盖 `macvlan` 及其对你的 LAN 或 WAN 上 IP 寻址的重要性。
* **第 7 章：容器配置选项** 简要介绍了一些基本的容器配置选项，并提供了修改配置选项的一些优势和副作用。
* **第 8 章：容器快照** 详细介绍了主服务器上容器的快照过程。
* **第 9 章：快照服务器** 介绍了快照服务器的设置和配置，以及如何创建主服务器与快照服务器之间的共生关系。
* **第 10 章：自动化快照** 介绍了快照创建的自动化以及将快照同步到快照服务器。
* **附录 A：工作站设置** 严格来说不属于生产服务器文档。它为那些希望在笔记本电脑或工作站上构建 Incus 容器实验环境的人提供了解决方案。

## 结论

你可以使用这些章节有效地设置企业级的主-快照 Incus 服务器对。在此过程中，你将学到大量关于 Incus 的知识。请注意，还有更多内容需要学习，将这些文档视为一个起点。

Incus 最重要的优势在于它在服务器上使用的经济性，允许快速启动操作系统安装，并允许在单台硬件上运行许多独立的应用程序服务器，从而最大限度地利用该硬件。
