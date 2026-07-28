---
title: 3 Incus 初始化与用户设置
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 9.4
tags:
  - incus
  - enterprise
  - incus initialization
  - incus setup
---

在本章的全部内容中，你需要是 root 用户或能够通过 `sudo` 成为 root。此外，我们假设你已经按照[第 2 章](02-zfs_setup.md)所述设置了 ZFS 存储池。如果你选择不使用 ZFS，可以使用其他存储池，但你需要调整初始化问题和答案。

## Incus 初始化

你的服务器环境已全部设置完毕。你已经准备好初始化 Incus。这是一个自动化脚本，会询问一系列问题以使你的 Incus 实例启动并运行：

```bash
incus admin init
```

以下是脚本的问题和我们的答案，并在必要时附上一些解释：

```text
Would you like to use clustering? (yes/no) [default=no]:
```

如果对集群感兴趣，请在[此处](https://linuxcontainers.org/incus/docs/main/explanation/clustering/)进行一些额外的研究。

```text
Do you want to configure a new storage pool? (yes/no) [default=yes]:
```

这似乎违反直觉。你已经创建了 ZFS 池，但这将在后面的问题中变得清晰。接受默认值。

```text
Name of the new storage pool [default=default]: storage
```

保留 "default" 是一个选项，但为了清晰起见，最好使用你给 ZFS 池起的相同名称。

```text
Name of the storage backend to use (btrfs, dir, lvm, zfs, ceph) [default=zfs]:
```

你需要接受默认值。

```text
Create a new ZFS pool? (yes/no) [default=yes]: no
```

这就是前面关于创建存储池的问题得到解决的地方。

```text
Name of the existing ZFS pool or dataset: storage
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

这是快照服务器所必需的。

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

## 设置用户权限

在继续之前，你必须创建 "incusadmin" 用户并确保其具有必要的权限。你需要 "incusadmin" 用户能够通过 `sudo` 提升为 root，并且需要它是 `incus-admin` 组的成员。要添加用户并确保它是两个组的成员，请执行以下操作：

```bash
useradd -G wheel,incus-admin incusadmin
```

设置密码：

```bash
passwd incusadmin
```

与其他密码一样，将其保存到安全位置。

## 设置 `root` 的 `subuid` 和 `subgid` 值

你必须设置 root 用户的 `subuid` 和 `subgid`（从属用户 ID 和组 ID 的范围）的值。此值应为：

```bash
root:1000000:1000000000
```

为此，编辑 `/etc/subuid` 并添加该行。完成后，你的文件将为：

```bash
root:1000000:1000000000
```

编辑 `/etc/subgid` 文件并添加该行。完成后，你的文件将为：

```bash
incusadmin:100000:65536
root:1000000:1000000000
```

在继续之前重启服务器。
