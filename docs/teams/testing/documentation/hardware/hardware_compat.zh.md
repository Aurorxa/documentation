---
title: 硬件兼容性
author: Chris Stackpole
contributors: Steven Spencer
translators:
---

## 历史

人们常常会问："Rocky 能在我的机器上安装吗？" 这是一个重要的问题，通常的答案是"可以"。然而，当问题变成"Rocky 能否与这个设备或设置配合工作？"时，挑战就更大了。在项目初期，测试团队需要尽可能多的报告来确认 Rocky 确实可以在广泛的硬件上安装。`xsos` 收集了这些信息并将其放入了[此 Git 仓库](https://github.com/rocky-linux/testing/tree/main/test-reports)。

在当时，它运行良好，但贡献困难，且难以搜索和筛选。这些报告已多年没有更新。

然而，"Rocky 能否在...上安装？" 这类问题仍然是有效的问题，因为新硬件不断发布。因此，人们希望帮助构建一个用户可以搜索和贡献的数据库。

## Hardware Probe 项目

[Hardware Probe 项目](https://github.com/linuxhw/hw-probe) 旨在回答这个问题。它是一个成熟的项目，拥有[一个网站](https://linux-hardware.org)，用户可以搜索硬件和系统。此外，有多种方式可以安装[该探针](https://linux-hardware.org/?view=howto)并轻松匿名提交结果。`hw-probe` 工具分析并收集来自多种实用工具（如 `lspci`、`smartctl` 和 `hwinfo`）的输出，以验证硬件并在提交到数据库之前检查驱动程序的正确加载和最佳功能。

以下是一个包含多个对[测试团队目的](https://linux-hardware.org/?probe=edebdf0568)有用项目的系统示例。

## 为何贡献

测试团队希望你为 Hardware Probe 项目数据库贡献你的系统，原因有三：

1. 无论你运行的是 Rocky 还是其他发行版，帮助 Hardware Probe 项目有助于所有 Linux 用户了解他们的硬件是否受支持并在 Linux 下工作。这对更广泛的社区有益。

2. 它为一个社区构建数据库，这些社区的用户可能有"Rocky 能否在...上安装？" 这样的问题。将你的 `hw-probe` 结果贡献到数据库有助于回答这个问题。变化和贡献越多，数据库对其他使用基于 EL 的发行版（如 Rocky Linux）的用户就越有帮助。

3. 有时驱动程序在上游会发生变更，甚至完全被移除。这有助于 Rocky 测试团队了解并更快地解决硬件问题，特别是那些驱动程序本应工作但由于某些原因在新版本中不再起作用的硬件。如果某硬件以前工作但现在不再工作，我们可能无法访问该硬件，这使得查找发布说明（如果硬件已被弃用）或提交上游错误报告（如果应受支持）变得困难。测试团队可用的硬件历史记录越多，帮助解决问题就越容易、越快。

## 如何贡献

对 Rocky Linux 来说幸运的是，`hw-probe` 工具在 EPEL 中可用，但 Rocky Linux 10 除外（尚未）。这不是一份关于如何使用该工具的全面指南。该工具有许多内置的有用功能，以下未涵盖。
这里我们只介绍安装和上传到数据库所需的功能。

### Rocky 8 和 Rocky 9

如果尚未安装 EPEL，请安装：

```bash
sudo dnf install -y epel-release
```

安装 `hw-probe`：

```bash
sudo dnf install -y hw-probe
```

请注意，EPEL 软件包并非总能获取到依赖项。

对于所有版本，你需要安装以下软件包：

```bash
sudo dnf install -y tar curl
```

在 Rocky 8 上，还需要安装 `xorg-x11-utils` 以获取 edid-decode [注意：此软件包不适用于 9/10 版本；请忽略警告]：

```bash
sudo dnf install -y xorg-x11-utils
```

运行探针并上传结果：

```bash
sudo -E hw-probe -all -upload
```

### Rocky 10

对于 Rocky 10，在撰写本文时，`hw-probe` 尚未进入主要 EPEL 仓库，但[有正在进行的构建工作](https://packages.fedoraproject.org/pkgs/hw-probe/hw-probe)。在此期间，获取 `hw-probe` 最简单的方式（依赖项包安装最少）是下载并使用 [AppImage](https://github.com/linuxhw/hw-probe/blob/master/README.md#appimage)。请查看链接获取最新版本。以下示例使用 `1.6.5-189`。

你需要安装 `libxcrypt-compat` 软件包：

```bash
sudo dnf install -y libxcrypt-compat
```

最后，使 AppImage 可执行：

```bash
chmod +x ./hw-probe-1.6.5-189-x86_64.AppImage
```

运行探针并上传结果：

```bash
sudo -E ./hw-probe-1.6.5-189-x86_64.AppImage -all -upload
```

### 通知测试团队

如果在官方发布版本上运行探针（在撰写本文时：8.10、9.7、10.1），请通过 chat.rockylinux.org（Mattermost 聊天）的 testing 频道将结果提交给测试团队。这仅是建议而非必需。这只是让测试团队知道你提交的硬件及其在当前 Rocky 中的状态。这对更广泛的社区和测试团队的历史记录（了解硬件在先前版本上是否运行）具有很大价值。

如果在候选发布版本（Release Candidate）或 Beta 版本上运行探针，无论状态如何，务必在 [chat.rockylinux.org](https://chat.rockylinux.org)（Mattermost 聊天）的 release 频道中报告链接。如果无法做到，请在你选择的社交媒体上直接私信 Rocky Linux，或在 [forums.rockylinux.org](https://forums.rockylinux.org) 的论坛中发帖。测试团队无法保证在 [chat.rockylinux.org](https://chat.rockylinux.org) Mattermost 频道之外的提交中你的名字会出现在致谢名单中，但我们将尽力包括所有提交者。

## 额外的贡献方式

如果能将 `hw-probe` 完全加入 EPEL 10，并支持尽可能多的架构，那将是极好的。任何有 EPEL 经验并愿意在此方面提供帮助的人都将受到极大的欢迎。你可以查看[上游错误报告](https://bugzilla.redhat.com/show_bug.cgi?id=2479630)或关注[测试团队的问题](https://github.com/rocky-linux/testing/issues/89)。

此外，任何愿意为 Hardware Probe 项目做出贡献的人都将提供巨大帮助。他们正在寻找能够协助编写测试和管理项目的人。

最后，任何愿意协助为测试团队设计一种能在不使上游数据库过载的情况下审查所有 Rocky 提交的好方法的人都将受到极大的欢迎。
