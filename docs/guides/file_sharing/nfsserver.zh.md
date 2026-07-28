---
title: NFS 服务器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - nfs
  - 存储
---

# NFS 服务器

## 引言

NFS（Network File System，网络文件系统）是一种分布式文件系统协议，允许您通过网络共享目录和文件。使用 NFS，远程用户和程序可以像访问本地文件一样访问远程系统上的文件。它对 Rocky Linux 环境中的集中存储管理非常有用。

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验

## 安装

安装 NFS 软件包：

```bash
dnf install nfs-utils
```

启用并启动 `nfs-server` 和 `rpcbind` 服务：

```bash
systemctl enable --now nfs-server rpcbind
```

验证 NFS 服务：

```bash
showmount -e localhost
```

## 配置

### 创建共享目录

```bash
mkdir -p /srv/nfs_share/shared
chown -R nfsnobody:nfsnobody /srv/nfs_share
chmod -R 755 /srv/nfs_share
```

### 配置导出(Exports)

编辑 `/etc/exports` 文件，添加共享目录：

```text
/srv/nfs_share/shared  192.168.0.0/24(rw,sync,no_root_squash)
```

其中：

* `rw` 读写访问权限
* `sync` 同步写入磁盘
* `no_root_squash` 允许 root 用户保留 root 权限

配置更改后，重新导出所有目录：

```bash
exportfs -arv
```

### 配置防火墙

使用 `firewalld` 启用 NFS 及相关服务：

```bash
firewall-cmd --add-service=nfs --permanent
firewall-cmd --add-service=rpc-bind --permanent
firewall-cmd --add-service=mountd --permanent
firewall-cmd --reload
```

或者，如果您使用的是 `iptables`：

```bash
iptables -A INPUT -p tcp --dport 111 -j ACCEPT
iptables -A INPUT -p udp --dport 111 -j ACCEPT
iptables -A INPUT -p tcp --dport 2049 -j ACCEPT
iptables -A INPUT -p udp --dport 2049 -j ACCEPT
service iptables save
```

## 挂载 NFS 共享

在客户端系统上，安装 `nfs-utils` 软件包：

```bash
dnf install nfs-utils
```

创建挂载点：

```bash
mkdir -p /mnt/nfs_share
```

挂载 NFS 共享：

```bash
mount -t nfs 192.168.0.10:/srv/nfs_share/shared /mnt/nfs_share
```

如果希望系统在启动时自动挂载 NFS 共享，将以下内容添加到 `/etc/fstab` 文件中：

```text
192.168.0.10:/srv/nfs_share/shared /mnt/nfs_share nfs defaults 0 0
```

## 测试

```bash
touch /mnt/nfs_share/testfile
```

在服务器上验证文件是否存在：

```bash
ls -l /srv/nfs_share/shared/testfile
```

## 结论

NFS 是 Linux 环境中共享文件和目录的简单而有效的解决方案。虽然它缺乏某些更现代的存储技术（如 GlusterFS）的可扩展性和冗余性，但它在小型网络环境中，对于集中管理和共享数据来说仍然非常可靠。
