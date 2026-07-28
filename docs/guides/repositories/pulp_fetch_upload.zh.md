---
title: Pulp 获取与分发 RPM 仓库
author: David Gomez
contributors: Steven Spencer, Ganna Zhyrnova 
tested with: 9.2
tags:
- Fetch
- Distribute
- RPM
- Repository
- Pulp
---

## 简介

使用 Rocky Linux 的开发者经常需要不同的远程 RPM 仓库来支持他们的工作。Pulp 是一个开源项目，可以通过帮助获取和分发不同的 RPM 仓库来满足开发者的这一需求。本指南展示了一个简单的示例，使用 Pulp 从 Rocky Linux vault 获取 BaseOS 和 AppStream。

## 要求

* 一个 Rocky Linux 系统
* 能够运行容器

## 设置 - 单容器

Pulp 有多种可能的设置方式，但为便于说明，本指南将使用单容器部署场景。为 Pulp 选择一个目录并创建以下目录和文件。

```bash
mkdir -p settings/certs pulp_storage pgsql containers
echo "CONTENT_ORIGIN='http://$(hostname):8080'" >> settings/settings.py
```

如果您启用了 SELinux（安全增强型 Linux），可以运行以下命令来部署 Pulp。如果未启用 SELinux，则可以从 `--volume` 行中移除 `:Z` 后缀：

```bash
$ podman run --detach \
             --publish 8080:80 \
             --name pulp \
             --volume "$(pwd)/settings":/etc/pulp:Z \
             --volume "$(pwd)/pulp_storage":/var/lib/pulp:Z \
             --volume "$(pwd)/pgsql":/var/lib/pgsql:Z \
             --volume "$(pwd)/containers":/var/lib/containers:Z \
             --device /dev/fuse \
             pulp/pulp
```

如果您浏览到 `http://localhost:8080/pulp/content/`，现在应该看到当前为空的 "/pulp/content/ 索引"。到本指南结束时，您将用您的仓库填充这些内容。

![空索引](images/empty_pulp_index.png)

## 创建 Pulp 远程

将 Pulp 远程视为远程源仓库。在本例中，远程源仓库是来自 Rocky Linux 9.2 vault 的 BaseOS 和 AppStream。您将使用这些远程来同步到您使用 Pulp 创建的仓库。有关远程策略的更多信息，请查看 [Pulp 官方文档](https://pulpproject.org/)。

```bash
pulp rpm remote create --name "rocky_92_appstream_vault" --url "https://dl.rockylinux.org/vault/rocky/9.2/AppStream/x86_64/os/" --policy on_demand
pulp rpm remote create --name "rocky_92_baseos_vault" --url "https://dl.rockylinux.org/vault/rocky/9.2/BaseOS/x86_64/os/" --policy on_demand
```

## Pulp 仓库副本

这些将是来自 Rocky Linux 9.2 vault 的 BaseOS 和 AppStream 的一对一仓库副本。如果您知道要用于同步仓库的远程，可以在创建仓库时添加这些远程。否则，如果您不知道要使用什么远程，或者这些远程可能会更改，那么您可以省略远程。对于本指南，远程的声明在创建仓库时进行。

```bash
pulp rpm repository create --name "R92_AppStream_Vault" --remote "rocky_92_appstream_vault"
pulp rpm repository create --name "R92_BaseOS_Vault" --remote "rocky_92_baseos_vault"
```

## Pulp 同步副本

!!! note

    添加 "--skip-type treeinfo" 很重要。否则，您最终会得到 BaseOS 和 AppStream 的奇怪混合，而不仅仅是 BaseOS 或 AppStream。这可能是由于仓库存在依赖闭环的问题。如果之前没有指定远程，您可以在这里添加。如果在创建时已添加了远程，则在同步时无需提及远程，因为它是隐含的。

```bash
pulp rpm repository sync --name "R92_AppStream_Vault" --skip-type treeinfo
pulp rpm repository sync --name "R92_BaseOS_Vault" --skip-type treeinfo
```

## Pulp 发布出版物

一旦您的仓库从远程同步完成，您将希望从这些仓库创建出版物以提供给分发。到目前为止，您仅使用远程和仓库的名称就能完成操作，但是 Pulp 也依赖于 `hrefs`，您可以互换使用它们。创建出版物后，请务必记下每个出版物的 `pulp_href` 值，因为它们在下一步中是必需的。

```bash
pulp rpm publication create --repository "R92_AppStream_Vault"
pulp rpm publication create --repository "R92_BaseOS_Vault"
```

## Pulp 创建分发

使用上一步出版物中的 `pulp_href`，您现在可以将该内容提供给一个分发。此内容随后将显示在 `http://localhost:8080/pulp/content/` 下，不再为空。您可以使用 `pulp rpm publication list` 再次检查出版物的 `pulp_href` 并查找 `pulp_href`。例如，下面是一个 BaseOS 的 `pulp_href`，但您的 `pulp_href` 可能不同，请相应地进行替换。

```bash
pulp rpm distribution create --name "Copy of BaseOS 92 RL Vault" --base-path "R92_BaseOS_Vault" --publication "/pulp/api/v3/publications/rpm/rpm/0195fdaa-a194-7e9d-a6a9-e6fd4eaa7a20/"
pulp rpm distribution create --name "Copy of AppStream 92 RL Vault" --base-path "R92_AppStream_Vault" --publication "<pulp_href>"
```

如果您查看 `http://localhost:8080/pulp/content/`，您应该看到您的两个仓库，它们是 Rocky Linux 9.2 AppStream 和 BaseOS vault 仓库的副本。

![content_index](images/pulp_index_content.png)

## 结论

Pulp 可以是一个非常通用的工具，用于获取多个仓库并根据需要进行分发。虽然这是一个基本示例，但您可以在更复杂和高级的各种部署场景中使用 Pulp。请查看[官方文档](https://pulpproject.org/)获取更多信息。
