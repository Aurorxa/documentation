---
title: openQA - openqa-cli POST 示例
author: Trevor Cooper
contributors: Lukas Magauer
tested_with:
tags:
  - testing
  - openQA
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
  level: Final
  r8: "8.10"
  r8date: 20250527
  r9: "9.6"
  r9date: 20250604
  r10: "10.0"
  r10date: 20250707
render_macros: true
---

本页面将简要介绍一些基本的 `openqa-cli` `POST` 示例。

## 系统/访问要求

要完成任一示例，请完成 [openQA - Rocky 生产环境访问](openqa_access.md) 文档中列出的 API `POST` 访问步骤。

## 基本 POST

基本的 `POST` 可用于 {{ rc.prod }} 提供的各种介质（media）的任一默认测试套件。以下示例展示了团队常用的一些标准 `POST`，并将用于演示一些细微的变化。

### FLAVOR=boot-iso

这第一个 `POST` 是最基本的，仅提供触发 {{ rc.prod }} {{ rc.r9 }} boot ISO 默认测试套件所需的最小变量集，在 `x86_64` 架构的 openqa worker 上运行。测试套件的所有测试均在 openQA 服务器上预先确定和配置。由于 boot ISO 不包含任何软件包，此测试套件实际上是从标准 {{ rc.prod }} 仓库服务器和/或镜像进行的网络安装。

```bash
$ openqa-cli api -X POST isos ISO=Rocky-{{ rc.r9 }}-x86_64-boot.iso ARCH=x86_64 \
  DISTRI=rocky FLAVOR=boot-iso VERSION={{ rc.r9}} CURRREL=9 BUILD={{ rc.r9date }}-Rocky-{{ rc.r9 }}-x86_64.0
```

### FLAVOR=minimal-iso

此 `POST` 演示了如何触发不同的介质类型（本例中为 minimal ISO）以及备选 {{ rc.prod }} 版本（本例中为 {{ rc.prod }} {{ rc.r8 }}）。从此 `POST` 和前一个 `POST` 可以看出，`BUILD` 变量通常指定测试套件的日期、版本和架构。由于 minimal ISO 包含执行 {{ rc.prod }} ***最小化***安装所需的所有软件包，这就是此测试套件的行为。

```bash
$ openqa-cli api -X POST isos ISO=Rocky-{{ rc.r8 }}-x86_64-minimal.iso ARCH=x86_64 \
  DISTRI=rocky FLAVOR=minimal-iso VERSION={{ rc.r8 }} CURRREL=8 BUILD={{ rc.r8date }}-Rocky-{{ rc.r8 }}-x86_64.0
```

### FLAVOR=package-set

此 `POST` 演示了最后一种常规介质类型（dvd ISO）的指定，以及所谓的 `FLAVOR`（本例中为 `package-set`）用于 `x86_64` 架构和 {{ rc.prod }} {{ rc.r9 }}。由于 dvd ISO 包含了特定版本 {{ rc.prod }} 发布时所有可用的软件包，`package-set` 测试套件将测试上述 `minimal-iso` 测试套件未包含的 {{ rc.prod }} 所有主要安装类型。

```bash
$ openqa-cli api -X POST isos ISO=Rocky-{{ rc.r9 }}-x86_64-dvd.iso ARCH=x86_64 \
  DISTRI=rocky FLAVOR=package-set VERSION={{ rc.r9 }} CURRREL=9 BUILD={{ rc.r9date }}-Rocky-{{ rc.r9 }}-x86_64.0
```

这三个测试套件为 {{ rc.prod }} 给定版本生成的所有 ISO 提供了最基本的测试。

## 高级 POST

除了上述 [基本 POST](#basic-post) 之外，还有使用 dvd ISO 介质并包含更多测试用例的额外默认测试套件。这些包括：

- 在图形界面、文本和串行控制台（serial console）中进行安装
- 标准 BIOS 和 UEFI 的安装
- Anaconda 帮助系统的验证
- 磁盘布局变种，包括 LVM、RAID、分区缩小和/或增大、iSCSI 和 LUKS
- 从各种网络来源进行 PXE 安装
- 各种语言的安装

这些测试套件的标准 `POST` 与上面的基本 POST 非常相似，如下所示...

### FLAVOR=dvd-iso

```bash
$ openqa-cli api -X POST isos ISO=Rocky-{{ rc.r9 }}-x86_64-dvd.iso ARCH=x86_64 \
  DISTRI=rocky FLAVOR=dvd-iso VERSION={{ rc.r9 }} CURRREL=9 BUILD={{ rc.r9date }}-Rocky-{{ rc.r9 }}-x86_64.0
```

### FLAVOR=universal

```bash
$ openqa-cli api -X POST isos ISO=Rocky-{{ rc.r9 }}-x86_64-dvd.iso ARCH=x86_64 \
  DISTRI=rocky FLAVOR=universal VERSION={{ rc.r9 }} CURRREL=9 BUILD={{ rc.r9date }}-Rocky-{{ rc.r9 }}-x86_64.0
```

## 按 BUILD 收集测试套件

openQA 的一个特性是，对于给定的作业组，使用相同 `BUILD` 标识符的测试套件会被收集到 Web UI 的单一视图中。

![openQA 主页视图...](../../../../assets/teams/testing/openqa_home_view.png){ loading=lazy }

因此，上面展示的所有使用 `BUILD={{ rc.r9date }}-Rocky-{{ rc.r9 }}-x86_64.0` 的示例都在单一视图中显示。此外，该视图可通过可预测的 URI 访问，例如 [`https://openqa.rockylinux.org/tests/overview?build={{ rc.r9date }}-Rocky-{{ rc.r9 }}-x86_64.0`](https://openqa.rockylinux.org/tests/overview?build=20250604-Rocky-9-x86_64.0)，如下所示...

![openQA 构建视图...](../../../../assets/teams/testing/openqa_build_view.png){ loading=lazy }

## 参考资料

[openQA 文档](https://open.qa/documentation/)

{% include 'teams/testing/content_bottom.md' %}
