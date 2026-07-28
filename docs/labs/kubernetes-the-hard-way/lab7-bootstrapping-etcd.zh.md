---
author: Wale Soyinka
contributors: Steven Spencer
tags:
  - kubernetes
  - k8s
  - lab exercise
  - etcd
---

# 实验 7: 引导 etcd 集群

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

Kubernetes 组件是无状态的，并将集群状态存储在 [etcd](https://github.com/etcd-io/etcd) 中。在本实验中，你将引导一个三节点 `etcd` 集群，并将其配置为实现高可用性和安全的远程访问。

## 前提条件

本实验中的命令必须在 `server` 实例上运行。使用 `ssh` 命令登录到 `server` 实例：

```bash
ssh root@server
```

## 引导 etcd 集群

所有 Kubernetes 组件将安装在 `/usr/local/bin` 目录下，并存储在 `jumpbox` 机器上。

### 下载并安装 etcd 二进制文件

从 `jumpbox` 获取 `etcd` 二进制文件：

```bash
scp root@jumpbox:~/kubernetes-the-hard-way/downloads/etcd-v3.4.36-linux-amd64.tar.gz .
```

创建 `/etc/etcd` 和 `/var/lib/etcd` 目录：

```bash
mkdir -p /etc/etcd /var/lib/etcd
chmod 700 /var/lib/etcd
```

提取 `etcd` 压缩文件，将 `etcd` 和 `etcdctl` 二进制文件复制到正确的目录并安装：

```bash
tar -xvf etcd-v3.4.36-linux-amd64.tar.gz
mv etcd-v3.4.36-linux-amd64/etcd* /usr/local/bin/
```

### 配置 etcd Server

```bash
  cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

`etcd` 正在集群内部 IP 地址上运行。将 `INTERNAL_IP` 环境变量设置为 `server` 实例的 IP 地址：

```bash
INTERNAL_IP=$(grep server machines.txt | cut -d " " -f 1)
```

设置 `etcd` 名称以匹配当前实例的主机名：

```bash
ETCD_NAME=server
```

创建 `etcd.service` systemd 单元文件：

```bash
cat <<EOF | tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/etcd-io

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster server=https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### 启动 etcd Server

```bash
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
```

## 验证

列出 `etcd` 集群成员：

```bash
etcdctl member list \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/kubernetes.crt \
  --key=/etc/etcd/kubernetes.key
```

```text
45c8020fc831ac2e, started, server, https://XXX.XXX.XXX.XXX:2380, https://XXX.XXX.XXX.XXX:2379, false
```

下一步: [引导 Kubernetes 控制平面](lab8-bootstrapping-kubernetes-controllers.md)
