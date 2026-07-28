---
title: 1 安装与配置
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd install
---

# 第 1 章：安装与配置

在本章中，你需要以 root 用户身份操作，或者能够通过 *sudo* 成为 root。

## 安装 EPEL 和 OpenZFS 仓库

LXD 需要 EPEL（Extra Packages for Enterprise Linux，企业 Linux 额外软件包）仓库，安装非常简单：

```bash
dnf install epel-release
```

安装完成后，检查是否有更新：

```bash
dnf upgrade
```

如果升级过程中有内核更新，请重启服务器。

### OpenZFS 仓库

安装 OpenZFS 仓库：

```bash
dnf install https://zfsonlinux.org/epel/zfs-release-2-8$(rpm --eval "%{dist}").noarch.rpm
```

## 安装 `snapd`、`dkms`、`vim` 和 `kernel-devel`

在 Rocky Linux 上安装 LXD 需要 snap 包。因此，需要安装 `snapd`（以及其他一些有用的程序）：

```bash
dnf install snapd dkms vim kernel-devel
```

现在启用并启动 snapd：

```bash
systemctl enable snapd
```

然后运行：

```bash
systemctl start snapd
```

在继续之前重启服务器。

## 安装 LXD

安装 LXD 需要使用 snap 命令。此时只是安装，尚未进行设置：

```bash
snap install lxd
```

## 安装 OpenZFS

```bash
dnf install zfs
```

## 环境设置

大多数服务器内核设置不足以运行大量容器。如果你从一开始就假设将在生产环境中使用服务器，则需要预先进行这些更改，以避免出现诸如 "Too many open files" 等错误。

幸运的是，通过一些文件修改和重启来调整 LXD 的设置并不困难。

### 修改 `limits.conf`

第一个需要更改的文件是 `limits.conf` 文件。此文件是自文档化的。请检查文件中注释的说明，以了解此文件的功能。要修改，请输入：

```bash
vi /etc/security/limits.conf
```

此文件全部由注释组成，底部显示了当前的默认设置。在文件结束标记（#End of file）上方的空白区域，你需要添加自定义设置。完成后的文件末尾将如下所示：

```text
# Modifications made for LXD

*               soft    nofile           1048576
*               hard    nofile           1048576
root            soft    nofile           1048576
root            hard    nofile           1048576
*               soft    memlock          unlimited
*               hard    memlock          unlimited
```

保存更改并退出（对于 *vi* 使用 ++shift+colon+"w"+"q"+exclam++）。

### 使用 `90-lxd.override.conf` 修改 sysctl.conf

通过 *systemd*，你可以在**不**修改主配置文件的情况下，更改系统整体配置和内核选项。而是将设置放在一个单独的文件中，该文件会覆盖你需要的特定设置。

要进行这些内核更改，你将在 `/etc/sysctl.d` 中创建一个名为 `90-lxd-override.conf` 的文件。输入：

```bash
vi /etc/sysctl.d/90-lxd-override.conf
```

!!! warning "RL 9 和 `net.core.bpf_jit_limit` 的最大值"

    由于最近的内核安全更新，`net.core.bpf_jit_limit` 的最大值似乎为 1000000000。如果你运行的是 Rocky Linux 9.x，请在下面的自文档化文件中调整此值。如果设置超过此限制**或**根本不设置，它将默认使用系统默认值 264241152，这在运行大量容器时可能不够。

将以下内容放入该文件中。请注意，如果你想知道在做什么，文件内容是自文档化的：

```bash
## The following changes have been made for LXD ##

# fs.inotify.max_queued_events specifies an upper limit on the number of events that can be queued to the corresponding inotify instance
 - (default is 16384)

fs.inotify.max_queued_events = 1048576

# fs.inotify.max_user_instances This specifies an upper limit on the number of inotify instances that can be created per real user ID -
(default value is 128)

fs.inotify.max_user_instances = 1048576

# fs.inotify.max_user_watches specifies an upper limit on the number of watches that can be created per real user ID - (default is 8192)

fs.inotify.max_user_watches = 1048576

# vm.max_map_count contains the maximum number of memory map areas a process may have. Memory map areas are used as a side-effect of cal
ling malloc, directly by mmap and mprotect, and also when loading shared libraries - (default is 65530)

vm.max_map_count = 262144

# kernel.dmesg_restrict denies container access to the messages in the kernel ring buffer. Please note that this also will deny access t
o non-root users on the host system - (default is 0)

kernel.dmesg_restrict = 1

# This is the maximum number of entries in ARP table (IPv4). You should increase this if you create over 1024 containers.

net.ipv4.neigh.default.gc_thresh3 = 8192

# This is the maximum number of entries in ARP table (IPv6). You should increase this if you plan to create over 1024 containers.Not nee
ded if not using IPv6, but...

net.ipv6.neigh.default.gc_thresh3 = 8192

# This is a limit on the size of eBPF JIT allocations which is usually set to PAGE_SIZE * 40000. Set this to 1000000000 if you are running Rocky Linux 9.x

net.core.bpf_jit_limit = 3000000000

# This is the maximum number of keys a non-root user can use, should be higher than the number of containers

kernel.keys.maxkeys = 2000

# This is the maximum size of the keyring non-root users can use

kernel.keys.maxbytes = 2000000

# This is the maximum number of concurrent async I/O operations. You might need to increase it further if you have a lot of workloads th
at use the AIO subsystem (e.g. MySQL)

fs.aio-max-nr = 524288
```

保存更改并退出。

此时重启服务器。

### 检查 *sysctl.conf* 值

重启后，以 root 用户身份重新登录服务器。需要检查覆盖文件是否确实完成了任务。

这并不难。不需要验证每个设置，但检查几个将验证设置是否已更改。使用 `sysctl` 命令：

```bash
sysctl net.core.bpf_jit_limit
```

将显示：

```bash
net.core.bpf_jit_limit = 3000000000
```

对覆盖文件中的其他几个设置执行相同的操作以验证更改。
