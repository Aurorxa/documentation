---
title: 附录 A - 工作站设置
author: Steven Spencer
contributors: Ganna Zhyrnova
tested_with: 9.4
tags:
  - incus
  - workstation
---

# 附录 A - 工作站设置

虽然不属于 Incus 服务器章节的一部分，但此过程将帮助那些希望在 Rocky Linux 工作站或笔记本电脑上拥有实验环境或半永久的操作系统和应用程序运行的人。

## 前提条件

* 在命令行操作方面得心应手
* 能够熟练使用命令行编辑器，如 `vi` 或 `nano`
* 需要每天或按需使用的稳定测试环境
* 能够成为 root 或具有 `sudo` 权限

## 安装

在命令行中，安装 EPEL 仓库：

```bash
sudo dnf install epel-release -y
```

安装完成后，进行升级：

```bash
sudo dnf upgrade
```

安装其他仓库：

```bash
sudo dnf config-manager --enable crb
sudo dnf copr enable neil/incus
```

安装一些必要的软件包：

```bash
sudo dnf install dkms vim kernel-devel bash-completion
```

安装并启用 Incus：

```bash
sudo dnf install incus incus-tools
sudo systemctl enable incus
```

在继续之前请重启你的笔记本电脑或工作站。

## Incus 初始化

如果你已经阅读了生产服务器章节，这与生产服务器初始化过程几乎完全相同。

```bash
sudo incus admin init
```

这将启动一个问答对话。

以下是脚本的问题和我们的答案，并在必要时附上一些解释：

```text
Would you like to use clustering? (yes/no) [default=no]: no
Do you want to configure a new storage pool? (yes/no) [default=yes]: yes
Name of the new storage pool [default=default]: storage
```

你可以选择接受默认值。

```text
Name of the storage backend to use (btrfs, dir, lvm, ceph) [default=btrfs]: dir
```

请注意，`dir` 比 `zfs` 稍慢。如果你可以留出一个空磁盘，你可以将该设备（例如：/dev/sdb）用于 `zfs` 设备，然后选择 `zfs`。

```text
Would you like to connect to a MAAS server? (yes/no) [default=no]:
```

Metal As A Service (MAAS) 超出了本文档的范围。

```text
Would you like to create a new local network bridge? (yes/no) [default=yes]:
What should the new bridge be called? [default=incusbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]:
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
```

你可以打开此选项以在你的 Incus 容器上使用 IPv6。

```text
Would you like the Incus server to be available over the network? (yes/no) [default=no]: yes
```

这是快照工作站所必需的。在此回答 "yes"。

```text
Address to bind Incus to (not including port) [default=all]:
Port to bind Incus to [default=8443]:
Trust password for new clients:
Again:
```

此信任密码是你连接或备份到快照服务器的方式。使用适合你环境的内容进行设置。将此条目保存到安全位置，例如密码管理器。

```text
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]
Would you like a YAML "incus admin init" preseed to be printed? (yes/no) [default=no]:
```

## 用户权限

接下来你需要做的是将你的用户添加到 `incus-admin` 组。同样，你需要为此使用 `sudo` 或是 root：

```text
sudo usermod -a -G incus-admin [username]
```

其中 [username] 是你在系统上的用户。

## 设置 `root` 的 `subuid` 和 `subgid` 值

你必须设置 root 用户的 `subuid` 和 `subgid`（从属用户 ID 和组 ID 的范围）的值。此值应为：

```bash
root:1000000:1000000000
```

为此，编辑 `/etc/subuid` 并添加该行。完成后，你的文件将为：

```bash
root:1000000:1000000000
```

将该行再次添加到 `/etc/subgid` 文件。你的文件将类似于：

```bash
incusadmin:100000:65536
root:1000000:1000000000
```

此时你已经做了很多更改。在继续之前，重启你的机器。

## 验证安装

要确保 `incus` 已启动并且你的用户具有权限，从 shell 提示符执行：

```text
incus list
```

注意你在此没有使用 `sudo`。你的用户可以输入这些命令。你将看到类似以下内容：

```bash
+------------+---------+----------------------+------+-----------+-----------+
|    NAME    |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+------------+---------+----------------------+------+-----------+-----------+
```

如果你看到了，那么一切顺利！

## 其余部分

从这一点开始，你可以使用我们 "Incus 生产服务器" 的章节继续。在工作站设置中，有一些事情你不需要那么关注。以下是你入门的推荐章节：

* [第 5 章 - 镜像的设置与管理](05-incus_images.md)
* [第 6 章 - 配置文件](06-profiles.md)
* [第 8 章 - 容器快照](08-snapshots.md)

## 进一步阅读

* [Incus 官方概述与文档](https://linuxcontainers.org/incus/docs/main/)

## 结论

Incus 是一个强大的工具，可在工作站或服务器上提高生产力。它非常适合工作站上的实验测试，并且还可以在其自己的私有空间中保持操作系统和应用程序的半永久实例可用。
