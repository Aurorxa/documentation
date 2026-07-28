---
title: 附录 A - 工作站设置
author: Steven Spencer
contributors: Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - lxd
  - workstation
---

# 附录 A - 工作站设置

虽然不是 LXD 服务器章节的一部分，但此过程将帮助那些希望在 Rocky Linux 工作站或笔记本上运行实验环境，或者运行半永久的操作系统和应用程序的人。

## 前提条件

* 习惯使用命令行
* 能够熟练使用命令行编辑器，如 `vi` 或 `nano`
* 愿意安装 `snapd` 来安装 LXD
* 需要每天或按需使用的稳定测试环境
* 能够成为 root 或拥有 `sudo` 权限

## 安装

从命令行安装 EPEL 仓库：

```bash
sudo dnf install epel-release 
```

安装完成后，进行升级：

```bash
sudo dnf upgrade
```

安装 `snapd`

```bash
sudo dnf install snapd 
```

启用 `snapd` 服务

```bash
sudo systemctl enable snapd
```

重启你的笔记本或工作站

安装 LXD 的 snap：

```bash
sudo snap install lxd
```

## LXD 初始化

如果你已阅读生产服务器章节，这里与生产服务器初始化过程几乎相同。

```bash
sudo lxd init
```

这将启动一个问答对话框。

以下是脚本的问题和我们的答案，并附有适当的说明：

```text
Would you like to use LXD clustering? (yes/no) [default=no]:
```

如果对集群感兴趣，请在[Linux containers 此处](https://documentation.ubuntu.com/lxd/en/latest/clustering/)进行额外研究。

```text
Do you want to configure a new storage pool? (yes/no) [default=yes]:
Name of the new storage pool [default=default]: storage
```

可选地，你可以接受默认值。

```text
Name of the storage backend to use (btrfs, dir, lvm, ceph) [default=btrfs]: dir
```

请注意，`dir` 比 `btrfs` 稍慢。如果你有先见之明留出一个空磁盘，你可以将该设备（例如：/dev/sdb）用于 `btrfs` 设备，然后选择 `btrfs`，但前提是你的宿主机操作系统支持 `btrfs`。Rocky Linux 和任何 RHEL 克隆版不支持 `btrfs` —— 至少目前还不支持。对于实验环境，`dir` 将正常工作。

```text
Would you like to connect to a MAAS server? (yes/no) [default=no]:
```

MAAS（Metal As A Service，裸金属即服务）超出了本文档的范围。

```text
Would you like to create a new local network bridge? (yes/no) [default=yes]:
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, "auto" or "none") [default=auto]:
What IPv6 address should be used? (CIDR subnet notation, "auto" or "none") [default=auto]: none
```

如果你想在 LXD 容器上使用 IPv6，可以开启此选项。这完全由你决定。

```text
Would you like the LXD server to be available over the network? (yes/no) [default=no]: yes
```

这对工作站快照来说是必需的。在此处回答 "yes"。

```text
Address to bind LXD to (not including port) [default=all]:
Port to bind LXD to [default=8443]:
Trust password for new clients:
Again:
```

此信任密码是你连接快照服务器或从快照服务器回连的方式。设置一个在环境中合理的密码。将此条目保存到安全位置，如密码管理器。

```text
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
```

## 用户权限

接下来需要做的是将你的用户添加到 lxd 组。同样，你需要使用 `sudo` 或成为 root 来完成此操作：

```text
sudo usermod -a -G lxd [username]
```

其中 [username] 是你在系统上的用户。

此时，你已经做了很多更改。在继续之前，重启你的机器。

## 验证安装

要确保 `lxd` 已启动并且你的用户具有权限，在 shell 提示符下执行：

```text
lxc list
```

请注意，这里没有使用 `sudo`。你的用户有能力输入这些命令。你将看到类似以下内容：

```bash
+------------+---------+----------------------+------+-----------+-----------+
|    NAME    |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+------------+---------+----------------------+------+-----------+-----------+
```

如果是这样，那么一切顺利！

## 其余内容

从此时起，你可以使用 "LXD 生产服务器" 中的章节继续操作。不过，在工作站设置中，有些内容你不需要过多关注。以下是推荐的章节：

* [第 5 章 - 设置与管理镜像](05-lxd_images.md)
* [第 6 章 - 配置文件](06-profiles.md)
* [第 8 章 - 容器快照](08-snapshots.md)

## 延伸阅读

* [LXD 入门指南](../../guides/containers/lxd_web_servers.md)，它将帮助你开始高效地使用 LXD。
* [LXD 官方概述和文档](https://documentation.ubuntu.com/lxd/en/latest/)

## 总结

LXD 是一个强大的工具，你可以在工作站或服务器上使用它来提高生产力。在工作站上，它非常适合用于实验测试，但也可以在其自己的私有空间中保持操作系统的半永久性实例和应用程序的运行。
