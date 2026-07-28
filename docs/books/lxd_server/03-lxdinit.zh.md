---
title: 3 LXD 初始化与用户设置
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd initialization
  - lxd setup
---

# 第 3 章：LXD 初始化与用户设置

在本章中，你需要以 root 身份操作，或者能够通过 `sudo` 成为 root。此外，假设你已经按照[第 2 章](02-zfs_setup.md)所述设置了 ZFS 存储池。如果你选择不使用 ZFS，也可以使用其他存储池，但需要对初始化问答进行相应调整。

## LXD 初始化

你的服务器环境已全部设置完毕。现在准备初始化 LXD。这是一个自动脚本，会询问一系列问题以启动你的 LXD 实例：

```bash
lxd init
```

以下是脚本的问题和我们的答案，并附有适当的说明：

```text
Would you like to use LXD clustering? (yes/no) [default=no]:
```

如果对集群感兴趣，请在[此处](https://documentation.ubuntu.com/lxd/en/latest/clustering/)进行额外研究。

```text
Do you want to configure a new storage pool? (yes/no) [default=yes]:
```

这似乎与直觉相悖。你已经创建了 ZFS 存储池，但这将在后面的问题中得到澄清。接受默认值。

```text
Name of the new storage pool [default=default]: storage
```

将此项保留为 "default" 也是可以的，但为了清晰起见，使用与 ZFS 池相同的名称更好。

```text
Name of the storage backend to use (btrfs, dir, lvm, zfs, ceph) [default=zfs]:
```

接受默认值。

```text
Create a new ZFS pool? (yes/no) [default=yes]: no
```

这里解决了之前关于创建存储池的问题。

```text
Name of the existing ZFS pool or dataset: storage
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

这对快照服务器来说是必需的。

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

## 设置用户权限

在继续之前，需要创建 "lxdadmin" 用户并确保其拥有所需的权限。"lxdadmin" 用户需要能够 `sudo` 到 root，并且需要是 lxd 组的成员。要添加用户并确保其属于这两个组，执行：

```bash
useradd -G wheel,lxd lxdadmin
```

设置密码：

```bash
passwd lxdadmin
```

与其他密码一样，将其保存到安全位置。
