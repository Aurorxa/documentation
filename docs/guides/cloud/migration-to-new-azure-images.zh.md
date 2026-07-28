---
title: 迁移到新的 Azure 镜像
author: Neil Hanlon
contributors: Steven Spencer, Ganna Zhyrnova
tags:
    - cloud
    - azure
    - microsoft azure
    - deprecation
---

!!! info "旧发布账户镜像于 2024 年 4 月弃用"

    Microsoft 发布账户是一个 Azure marketplace 账户，允许开发者将其产品发布到 Microsoft AppSource 或 Azure Marketplace。
    RESF 在 Azure 中通过两个独立的发布账户提供 Rocky Linux 虚拟机镜像：一个旧账户，标识为 `erockyenterprisesoftwarefoundationinc1653071250513`，以及一个名为 `resf` 的较新的官方账户。
    在旧发布账户（`erockyenterprisesoftwarefoundationinc1653071250513`）下发布的镜像已于 2024 年 4 月 23 日标记为弃用，并有 180 天（6 个月）的过渡期，之后将无法再使用。

    要继续在 Azure 上使用 Rocky Linux，您必须按照本指南迁移到新的发布账户（`resf`）或新的社区画廊（Community Galleries）镜像。

# 迁移指南：过渡到 Azure 上的新 Rocky Linux 镜像

本指南提供了将 Azure 虚拟机（VM）从已弃用的 Rocky Linux 镜像迁移到 `resf` 发布商账户下的新镜像或使用社区画廊（Community Galleries）的详细步骤。遵循本指南将确保平稳过渡，最大限度减少中断。

## 开始之前

- 确保您拥有 VM 的最新备份。虽然迁移过程不应影响您的数据，但进行备份是任何系统更改的最佳实践。
- 验证您在 Azure 账户中拥有创建新 VM 和管理现有 VM 的必要权限。

## 第 1 步：定位现有的 VM

识别使用旧 Rocky Linux 镜像部署的 VM。您可以通过按旧发布商账户名称过滤您的 VM 来做到这一点：

```text
erockyenterprisesoftwarefoundationinc1653071250513`
```

## 第 2 步：准备新的 VM

1. **导航**到 Azure Marketplace。
2. **搜索** `resf` 发布商账户下的新 Rocky Linux 镜像，或访问社区画廊（Community Galleries）。
    - 当前 Marketplace 链接：
      - [Rocky Linux x86_64](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/resf.rockylinux-x86_64)
    - [Rocky Linux aarch64](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/resf.rockylinux-aarch64)
    - 访问社区画廊镜像的完整说明请参见此[新闻文章](https://rockylinux.org/news/rocky-on-azure-community-gallery/)
3. **选择**所需的 Rocky Linux 版本并**创建新的 VM**。在设置过程中，您可以选择与现有 VM 相同的 VM 大小和其他配置以确保兼容性。

## 第 3 步：传输数据

### 选项 A：使用 Azure 托管磁盘（推荐，简单）

1. **停止**您的现有 VM。
2. **分离**现有 VM 上的操作系统磁盘。
3. **附加**已分离的磁盘到新 VM 作为数据磁盘。
4. **启动**新 VM。如果需要，您可以挂载旧操作系统磁盘并将数据复制到新磁盘。

### 选项 B：使用数据传输工具（适用于复杂环境或特定需求）

1. **选择**数据传输工具，如 `rsync` 或 Azure Blob Storage 用于传输数据。
2. **传输**数据从旧 VM 到新 VM。这可能包括应用程序数据、配置和用户数据。

```bash
# rsync 命令示例
rsync -avzh /path/to/old_VM_data/ user@new_VM_IP:/path/to/new_VM_destination/
```

## 第 4 步：重新配置新 VM

1. **重新应用**您在旧 VM 上拥有的任何自定义配置或调整到新 VM，确保其与预期的环境设置匹配。
2. **测试**新 VM 以确认应用程序和服务按预期运行。

## 第 5 步：更新 DNS 记录（如果适用）

如果您通过特定域名访问您的 VM，必须更新您的 DNS 记录以指向新 VM 的 IP 地址。

## 第 6 步：停用旧 VM

一旦确认新 VM 运行正常并且已移动所有必要的数据和配置，您可以**解除分配并删除**旧 VM。

## 最终步骤

- 验证新 VM 上的所有服务按预期运行。
- 监控新 VM 的性能和健康状况，确保其满足您的需求。

## 支持

如果您在迁移过程中遇到任何问题或有疑问，可以获得支持。请访问 [Rocky Linux 支持渠道](https://wiki.rockylinux.org/rocky/support/) 寻求帮助。
