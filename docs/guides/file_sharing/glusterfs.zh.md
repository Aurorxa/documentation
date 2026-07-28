---
title: GlusterFS 集群
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - glusterfs
  - 存储
  - 集群
---

# GlusterFS 集群配置

## 引言

GlusterFS 是一个可扩展的开源网络文件系统(network filesystem)，能够将来自多个服务器的不同存储资源整合到一个统一的全局命名空间中。GlusterFS 通常用作企业应用的存储后端，支持高可用和高性能。它通过将本地文件系统聚合并通过 InfiniBand RDMA 或 TCP/IP 互联，提供单一全局命名空间来实现这一切。

GlusterFS 可用于在多个服务器之间创建冗余存储(redundant storage)，无需昂贵的专用存储硬件。本指南将引导您在 Rocky Linux 上配置一个 GlusterFS 集群。

## 前提条件

* 至少两台运行 Rocky Linux 的服务器，每台服务器都有独立的附加磁盘或分区用于存储
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验

## 准备系统

所有节点都需要进行相互解析。为每个节点配置主机名并在所有节点上添加 `/etc/hosts` 条目：

```text
192.168.0.10   gluster-node1.example.com   gluster-node1
192.168.0.11   gluster-node2.example.com   gluster-node2
192.168.0.12   gluster-node3.example.com   gluster-node3
```

在所有节点上安装 GlusterFS：

```bash
dnf install glusterfs-server
```

在所有节点上启动并启用服务：

```bash
systemctl enable --now glusterd
```

## 配置磁盘

在每台服务器的专用磁盘上创建分区并挂载。

假设磁盘为 `/dev/sdb`，使用 `parted` 创建分区：

```bash
parted /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary xfs 1MiB 100%
```

格式化分区：

```bash
mkfs.xfs -f -i size=512 /dev/sdb1
```

创建挂载点并挂载：

```bash
mkdir -p /data/glusterfs
mount /dev/sdb1 /data/glusterfs
```

在 `/etc/fstab` 中添加条目以确保在重启后挂载：

```bash
echo '/dev/sdb1 /data/glusterfs xfs defaults 0 0' >> /etc/fstab
```

创建 brick 目录：

```bash
mkdir -p /data/glusterfs/brick1
```

## 防火墙配置

在 `firewalld` 中启用 GlusterFS 服务：

```bash
firewall-cmd --add-service=glusterfs --permanent
firewall-cmd --reload
```

## 创建可信存储池

从第一台节点(`gluster-node1`)将其他节点添加到存储池中：

```bash
gluster peer probe gluster-node2
gluster peer probe gluster-node3
```

验证池状态：

```bash
gluster pool list
```

输出应显示所有节点为 `Connected` 状态。

## 创建卷(Volume)

在所有节点上创建所需的 brick 目录：

```bash
mkdir -p /data/glusterfs/brick1
```

创建 GlusterFS 卷：

```bash
gluster volume create gv0 replica 3 transport tcp \
gluster-node1:/data/glusterfs/brick1 \
gluster-node2:/data/glusterfs/brick1 \
gluster-node3:/data/glusterfs/brick1
```

此命令创建一个名为 `gv0` 的卷，三台服务器之间的复制因子为 3。

启动卷：

```bash
gluster volume start gv0
```

验证卷状态：

```bash
gluster volume info
```

## 挂载卷

客户端可以使用 GlusterFS 的原生客户端或 NFS 进行挂载。

### 安装 GlusterFS 客户端软件

```bash
dnf install glusterfs-fuse
```

### 挂载卷

创建一个挂载点并挂载卷：

```bash
mkdir -p /mnt/glusterfs
mount -t glusterfs gluster-node1:/gv0 /mnt/glusterfs
```

如果希望自动挂载，请将以下内容添加到 `/etc/fstab` 文件中：

```bash
gluster-node1:/gv0 /mnt/glusterfs glusterfs defaults,_netdev 0 0
```

## 测试

通过创建文件并检查其是否在所有节点上可见来测试卷：

```bash
touch /mnt/glusterfs/testfile
```

您应该能在所有其他节点的对应目录中看到该文件。

## 结论

GlusterFS 提供了可扩展的、去中心化的分布式存储。它可以轻松扩展以满足不断增长的存储需求。通过其在多台服务器上轻松复制数据的能力，GlusterFS 可以被用作高可用数据存储解决方案的基础。
