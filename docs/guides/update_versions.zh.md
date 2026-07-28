---
title: Rocky 支持的版本升级
author: Steven Spencer
contributors: Ganna Zhyrnova
---

**或者** 如何复制任何 Rocky 机器。

## 简介

自 Rocky Linux 项目第一天起，就有人问：==如何从 CentOS 7 升级到 Rocky 8，或从 Rocky 8 升级到 Rocky 9？== 答案总是一样的：**该项目不支持从一个主版本就地升级到另一个主版本。您需要重新安装以移动到下一个主版本。** 明确地说，这**是**正确答案。本文档允许用户从一个主版本移动到下一个主版本，使用正确的 Rocky 支持的全新安装过程。您可以使用此方法重建相同的 Rocky Linux 版本。例如，将 9.5 全新安装为带有所有软件包的 9.5。

!!! note "注意事项"

    即使有此过程，在从一个操作系统（OS）的旧版本升级到相同或不同操作系统的较新版本时，许多事情可能会出错。程序变得过时，并被维护人员以完全不同的软件包名称替换，或者名称从一个操作系统到另一个操作系统不匹配。此外，请了解您机器的软件仓库，并验证它们在新操作系统下仍然可用。如果从非常旧的版本迁移到非常新的版本，请确保您的 CPU 和其他机器要求与新版本匹配。由于这些和其他许多不同的原因，您必须保持谨慎，并注意在执行此过程中出现的任何错误或问题。在这里，作者使用了 Rocky Linux 8 作为旧版本，Rocky Linux 9 作为新的主版本。所有示例的表述都使用这两个版本。您随时可以自行决定操作并承担风险。

## 步骤摘要

1. 从旧安装中获取用户列表（`userid.txt`）。
2. 从旧安装中获取仓库列表（`repolist.txt`）。
3. 从旧安装中获取软件包列表（`installed.txt`）。
4. 将所有数据、配置、实用工具和脚本从旧安装备份到非易失性位置，以及创建的 `.txt` 文件。
5. 验证您要安装的硬件支持您正在安装的操作系统。（CPU、内存、磁盘空间等。）
6. 在硬件上执行您使用的操作系统的全新安装。
7. 执行 `dnf upgrade` 以获取自 ISO 文件创建以来可能已更新的任何软件包。  
8. 通过检查 `userid.txt` 文件创建任何需要的用户。
9. 安装 `repolist.txt` 文件中与 Rocky 无关的任何缺失仓库。（有关 EPEL 和 Code Ready Builder（CRB）仓库的注意事项。）
10. 使用 `installed.txt` 文件的过程安装软件包。

## 步骤详情

!!! info "相同版本升级"

    正如前面讨论的，此过程应同样适用于使用相同操作系统版本（如 8.10 到 8.10 或 9.5 到 9.5）复制机器安装。不同的是，在从 `installed.txt` 文件安装软件包时您应该不需要 `--skip-broken`。如果在安装版本时遇到软件包错误，您可能缺少了一个仓库。停止该过程并重新检查 `repolist.txt` 文件。这里的示例使用 8.10 作为旧安装，9.5 作为新安装。

!!! warning "版本 10 未知"

    由于 9.5 和即将到来的版本 10 之间的巨大变化，此过程在 9.5 和 10 之间**可能无法工作**。对此的探索将在有 10 的发布版本进行测试时进行。 

### 示例旧机器

此处使用的旧机器是 Rocky Linux 8。安装包括几个 EPEL（Extra Packages for Enterprise Linux，企业 Linux 额外软件包）仓库软件包。

!!! info "Code Ready Builder"

    Rocky Linux 9 中的 Code Ready Builder (CRB) 仓库取代了版本 8 中已弃用的 PowerTools 仓库中的功能。如果从 8 版本移动到 9 版本且您有 EPEL，则需要在新机器上启用 CRB，执行以下操作：

    ```bash
    sudo dnf config-manager --enable crb
    ```

#### 获取用户列表

您需要手动在新机器上创建任何用户，因此您需要知道需要创建哪些用户账户。用户账户通常从用户 ID 1000 开始并递增。

```bash
sudo getent passwd > userid.txt
```

#### 获取仓库列表

您需要旧机器上存在的仓库列表：

```bash
sudo ls -al /etc/yum.repos.d/ > repolist.txt
```

#### 获取软件包列表

使用以下命令生成软件包列表：

```bash
sudo dnf list installed | awk 'NR>1 {print $1}' | sort -u > installed.txt
```

这里，`NR>1` 从列中消除记录一，该记录具有来自 `dnf list installed` 命令的 "Installed"。它不是软件包，因此您不需要它。`{print $1}` 表示只使用第一列。您不需要列出中的软件包版本或它来自的仓库。

您将不需要安装任何与内核相关的软件包。如果您省略此步骤，再次安装它们也没有关系。您可以使用以下命令删除内核行：

```bash
sudo sed -i '/kernel/d' installed.txt
```

#### 备份任何数据

这可以涵盖很多事情。确保您知道您正在替换的机器的用途及其所有程序组件（数据库、邮件服务器、DNS 等）。如果您有任何疑问，只需备份它。

#### 复制文件

将您创建的文本文件复制到非易失性位置和所有备份数据。

### 示例新机器

您的 Rocky Linux 9 全新安装已完成。您需要获取自 ISO 镜像创建以来的任何软件包更新：

```bash
sudo dnf upgrade
```

您已准备好开始从早期过程中存储它们的位置复制您的文本文件和备份。

#### 创建用户

检查 `userid.txt` 文件并在新机器上创建您需要的用户。

#### 安装仓库

检查 `repolist.txt` 文件并手动安装您需要的仓库。您可以忽略与 Rocky 相关的仓库。记住我们有来自 EPEL 的软件包，因此您将需要 CRB 仓库而不是 PowerTools：

```bash
sudo dnf config-manager --enable crb
```

安装 EPEL：

```bash
sudo dnf install epel-release
```

安装来自 `repolist.txt` 文件中不是基于 Rocky 或 EPEL 的任何其他仓库。

#### 安装软件包

仓库安装完成后，尝试从 `installed.txt` 安装您的软件包：  

```bash
sudo dnf -y install $(cat installed.txt)
```

无论您启用了哪些仓库，在 Rocky Linux 8 和 Rocky Linux 9 之间某些软件包将不存在。运行此命令可让您了解这些软件包是什么。

以下是作者测试机器上未安装的软件包（重新组织为一列而非长字符串）：

```text
Error: Unable to find a match: 
OpenEXR-libs.x86_64 
bind-export-libs.x86_64 
dhcp-libs.x86_64 
fontpackages-filesystem.noarch 
hardlink.x86_64 
ilmbase.x86_64 
libXxf86misc.x86_64 
libcroco.x86_64 
libmcpp.x86_64 
libreport-filesystem.x86_64 
mcpp.x86_64 
network-scripts.x86_64 
platform-python.x86_64 
platform-python-pip.noarch 
platform-python-setuptools.noarch 
xorg-x11-font-utils.x86_64
```

!!! note

    如果您在新安装上需要这些缺失软件包的功能，请将它们保存到文件中供以后使用。您可以使用以下命令查看缺失软件包的可用性状态：

    ```bash
    sudo dnf whatprovides [package_name]
    ```

重新运行该命令，但这次在末尾附加 `--skip-broken`：

```bash
sudo dnf -y install $(cat installed.txt) --skip-broken
```

由于您刚刚做了许多更改，您应该在继续之前重启。

#### 恢复您的备份

安装完所有软件包后，恢复您的备份、修改后的配置文件、脚本以及您在移动到新机器前备份的其他实用工具。

## 结论

没有（受 Rocky Linux 支持的）神奇程序可以从一个主版本移动到另一个主版本。Rocky Linux 开发者仅支持全新安装。这里提供的程序允许您从一个主版本移动到另一个主版本，同时遵循 Rocky 团队的最佳实践。

此过程假设是一个简单的安装。但是，如果您的安装是复杂的，您可能需要采取更多步骤。您可以将此过程作为指南使用。

## 免责声明

虽然基本文档是作者的，但 [Forum](https://forums.rockylinux.org/t/boot-too-small-rebuild/17415) 中的两位个人建议了一种更简洁的方式来生成 `installed.txt` 并消除了内核软件包。感谢所有为该过程提供意见的人。
