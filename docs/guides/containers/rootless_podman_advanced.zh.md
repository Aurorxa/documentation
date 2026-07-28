---
title: Rootless Podman 高级配置
author: Steven Spencer
contributors: Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - containers
  - podman
---

# Rootless Podman

## 前提条件

* 已在本地安装发行版的 Rocky Linux 工作站或服务器
* 对命令行和编辑器有一定了解（本文以 `vi` 为例，可替换为您喜欢的编辑器）
* 对 `sudo`、用户、组和权限有基本了解

## 引言

Podman 有两种主要的操作模式：root（作为系统守护进程运行）和 rootless（作为用户运行）。通常，默认运行时是 rootless 模式。这使得用户可以根据需要启动管理自己的容器，而无需担心系统服务。由此带来的好处是系统的安全性得到了显著提升。

本指南讨论的是 rootless 容器的**高级配置**。如果您还没有阅读[Podman 入门指南](podman_guide.md)，建议先阅读。本指南假设您已经熟悉 Podman 的配置，因此内容上会更为深入。

通过一些额外的步骤，您可以在容器与主机之间实现更为精细的权限控制。当然，这些步骤是可选的，但强烈建议进行设置。

## 要求

* rootless 容器——本流程假设您已根据[此 Podman 指南](podman_guide.md)安装了 Podman
* `slirp4netns` 软件包——Podman 依赖 `slirp4netns` 为 rootless 容器提供网络功能。在 Rocky Linux 上，该软件包应与 `podman` 同时安装
* `shadow-utils` 软件包（应已安装）

## 文件暴露

### `/etc/subuid` 和 `/etc/subgid`

`/etc/subuid` 和 `/etc/subgid` 是 Linux 用来管理用户命名空间(user namespaces)权限的文件，这些文件将用户和组的 ID 映射到一个从属(subordinate)范围。

`/etc/subuid` 和 `/etc/subgid` 文件的外观示例如下：

```text
testuser:200000:1000
```

这个配置的含义是：用户 `testuser` 被分配了一个从属的用户 ID 范围，起始 ID 为 200000，长度为 1000。因此，当使用用户命名空间的容器运行时，容器内部的 ID 将映射到主机系统上的 200000-201000 范围内的 ID。

对于 rootless 容器，通常这些文件的默认设置就足够了。但如果一个用户在主机系统上拥有大量文件，可能需要扩大其从属(subordinate) ID 的范围。

Podman 本身并不直接在 `subuid` 和 `subgid` 这两个文件中创建条目；相反，当您首次以 rootless 模式运行 Podman 容器时，它会自动通过将主机的用户 ID 映射到从属 ID，来填补容器内的用户 ID。这种自动映射机制确保了 rootless 容器安全高效地运行。

### 设置

要验证您的 rootless Podman 网络，请使用以下命令：

```bash
podman unshare cat /proc/self/uid_map
```

如果您的用户不存在从属(subordinate) ID 映射条目，您将看到如下输出：

```text
         0       1000          1
```

如果您看到此输出，说明容器内唯一的 root 用户能够与他们主机上的用户映射。如果这正是您所需要的，那么您就已经设置完毕了。

如果您需要扩大从属 ID 范围，请使用 `usermod` 命令：

```bash
usermod --add-subuids 200000-265535 --add-subgids 200000-265535 testuser
```

这将把您指定的范围添加到 `/etc/subuid` 和 `/etc/subgid` 文件中。您需要确保您添加的 ID 范围足够大，并且不会与其他现有范围重叠。完成后，您可以使用以下命令再次检查已分配的从属 ID 范围：

```bash
podman unshare cat /proc/self/uid_map
```

## 网络

您可能需要在 `containers.conf` 文件中添加一些内容。在 Rocky Linux 8.5 或 8.6 上，Podman 默认将 `containers.conf` 文件安装在 `/usr/share/containers/containers.conf`，但首次启动容器后，它会将其复制到 `/etc/containers/containers.conf`。因此，只需编辑 `/etc/containers/containers.conf` 文件即可。

在 Rocky Linux 9 上，该文件仅位于 `/usr/share/containers/containers.conf`，需要进行复制：

```bash
cp /usr/share/containers/containers.conf /etc/containers/containers.conf
```

将以下内容添加到该文件底部或默认的网络部分：

```text
default_rootless_network_cmd = "slirp4netns"
network_cmd_options = ["enable_ipv6"]
```

!!! note "网络选项说明"

    `network_cmd_options = ["enable_ipv6"]` 这一行用于启用 IPv6。根据您的需求，可以决定是否添加。

如果 `/etc/containers/containers.conf` 不存在，您可以按照上述说明复制 `root` 用户的文件并直接修改。

### 设置用户配置文件

首先，在您的 `$HOME/.config/containers/` 目录下为容器创建存储配置：

```bash
mkdir -p $HOME/.config/containers
cp /etc/containers/storage.conf /home/testuser/.config/containers/storage.conf
```

但是，您不能直接使用 `/etc/containers/storage.conf`，因为它包含一些过时的选项。运行以下命令以生成一个新的且不带过时选项的 `storage.conf` 文件：

```bash
sed -i -e 's|^#mount_program|mount_program|g' -e '/additionalimage/d' -e '/mountopt/d' /home/testuser/.config/containers/storage.conf
```

为了让 rootless 模式正常运行，您还需要增加 Podman 的用户命名空间(user namespace)数量。为此，您需要在 `/etc/sysctl.d/` 目录下创建一个新文件 `99-userns.conf`：

```bash
cat << EOF >> /etc/sysctl.d/99-userns.conf
user.max_user_namespaces=10000
EOF
```

使用 `sysctl` 重新加载配置以使其生效：

```bash
sysctl -p /etc/sysctl.d/99-userns.conf
```

如果您希望从您的 rootless Podman 端口能够被外部访问到（例如 `80` 或 `443`），则需要在主机上利用防火墙进行转发。

首先，找出您的主机接口：

```bash
nmcli connection show
```

连接信息显示后，使用 `nmcli` 命令修改 `my_connection` 网卡：

```bash
nmcli connection modify my_connection forward-port 80:8080
```

## 结论

虽然这些步骤对于简单的 rootless 容器运行来说并非绝对必需，但它们能够显著提升系统的安全性、稳定性和性能。如果您经常使用 rootless 容器，建议您执行这些配置步骤。
