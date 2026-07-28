---
title: 1 安装与配置
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 9.4
tags:
  - incus
  - enterprise
  - incus install
---

在本章的全部内容中，你需要是 root 用户或能够通过 *sudo* 提升为 root。

## 安装 EPEL 和 OpenZFS 仓库

Incus 需要 EPEL（Extra Packages for Enterprise Linux，企业级 Linux 额外软件包）仓库，可以通过以下命令轻松安装：

```bash
dnf install epel-release -y
```

安装后，验证是否有更新：

```bash
dnf upgrade
```

如果在升级过程中有内核更新，请重启服务器。

### OpenZFS 仓库

通过以下命令安装 OpenZFS 仓库：

```bash
dnf install https://zfsonlinux.org/epel/zfs-release-2-8$(rpm --eval "%{dist}").noarch.rpm
```

## 安装 `dkms`、`vim` 和 `kernel-devel`

安装一些必要的软件包：

```bash
dnf install dkms vim kernel-devel bash-completion
```

## 安装 Incus

你需要启用 CRB 仓库以获取一些特殊软件包，并启用 Neil Hanlon 的 COPR（Cool Other Package Repo）：

```bash
dnf config-manager --enable crb
dnf copr enable neil/incus
dnf install incus incus-tools
```

启用并启动服务：

```bash
systemctl enable incus --now
```

在继续之前重启服务器。

## 安装 OpenZFS

```bash
dnf install zfs
```

## 环境设置

运行许多容器需要比大多数服务器默认设置更多的内核设置。如果你从一开始就假设你将在生产环境中使用服务器，则需要提前进行这些更改，以避免诸如"打开文件过多"之类的错误。

幸运的是，通过一些文件修改和重启，调整 Incus 的设置并不难。

### 修改 `limits.conf`

首先要更改的文件是 `limits.conf` 文件。此文件是自文档化的。请检查文件中注释的解释以了解此文件的作用。要进行修改，请输入：

```bash
vi /etc/security/limits.conf
```

此整个文件由注释组成，并在底部显示当前默认设置。你需要在文件结束标记（#End of file）上方的空白区域添加你的自定义设置。完成后，文件末尾将如下所示：

```text
# Modifications made for LXD

*               soft    nofile           1048576
*               hard    nofile           1048576
root            soft    nofile           1048576
root            hard    nofile           1048576
*               soft    memlock          unlimited
*               hard    memlock          unlimited
```

保存更改并退出（对 *vi* 使用 ++shift+colon+"w"+"q"+exclam++）。

### 通过 `90-incus-override.conf` 修改 `sysctl.conf`

借助 *systemd*，你可以在*不*修改主配置文件的情况下更改系统的整体配置和内核选项。相反，将你的设置放在一个单独的文件中，该文件将覆盖你需要的特定设置。

要进行这些内核更改，你将在 `/etc/sysctl.d` 中创建一个名为 `90-incus-override.conf` 的文件。为此，输入以下命令：

```bash
vi /etc/sysctl.d/90-incus-override.conf
```

将以下内容放入该文件中。请注意，如果你想知道你在这里做什么，文件内容是自文档化的：

```bash
## The following changes have been made for LXD ##

# fs.inotify.max_queued_events 指定可排队到相应 inotify 实例的事件数量的上限 - （默认值为 16384）

fs.inotify.max_queued_events = 1048576

# fs.inotify.max_user_instances 指定每个真实用户 ID 可创建的 inotify 实例数量的上限 - （默认值为 128）

fs.inotify.max_user_instances = 1048576

# fs.inotify.max_user_watches 指定每个真实用户 ID 可创建的监视数量的上限 - （默认值为 8192）

fs.inotify.max_user_watches = 1048576

# vm.max_map_count 包含一个进程可能拥有的内存映射区域的最大数量。内存映射区域作为调用 malloc 的副作用、直接由 mmap 和 mprotect 使用，以及在加载共享库时使用 - （默认值为 65530）

vm.max_map_count = 262144

# kernel.dmesg_restrict 拒绝容器访问内核环形缓冲区中的消息。请注意，这也将拒绝主机系统上的非 root 用户访问 - （默认值为 0）

kernel.dmesg_restrict = 1

# 这是 ARP 表（IPv4）中的最大条目数。如果创建超过 1024 个容器，应该增加此值。

net.ipv4.neigh.default.gc_thresh3 = 8192

# 这是 ARP 表（IPv6）中的最大条目数。如果计划创建超过 1024 个容器，应该增加此值。如果不使用 IPv6 则不需要，但是……

net.ipv6.neigh.default.gc_thresh3 = 8192

# 这是对 eBPF JIT 分配大小的限制，通常设置为 PAGE_SIZE * 40000。如果你正在运行 Rocky Linux 9.x，请将其设置为 1000000000

net.core.bpf_jit_limit = 1000000000

# 这是非 root 用户可以使用的最大键数量，应该高于容器数量

kernel.keys.maxkeys = 2000

# 这是非 root 用户可以使用的密钥环的最大大小

kernel.keys.maxbytes = 2000000

# 这是并发异步 I/O 操作的最大数量。如果你有大量使用 AIO 子系统的工作负载（例如 MySQL），可能需要进一步增加此值

fs.aio-max-nr = 524288
```

保存更改并退出。

此时，重启服务器。

### 检查 `sysctl.conf` 的值

重启后，以 root 用户身份重新登录到服务器。你需要检查我们的覆盖文件是否确实完成了工作。

这并不难做到。除非你想，否则不需要验证每个设置，但检查几个设置将验证设置是否已更改。使用 `sysctl` 命令执行此操作：

```bash
sysctl net.core.bpf_jit_limit
```

这将显示：

```bash
net.core.bpf_jit_limit = 1000000000 
```

对覆盖文件中的其他几个设置执行相同操作以验证更改。
