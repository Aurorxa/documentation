---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - kubernetes
  - k8s
  - lab exercise
  - runc
  - containerd
  - etcd
  - kubectl
---

# 实验 11: 配置 Pod 网络路由

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

调度到节点的 Pod 从该节点的 Pod CIDR 范围获取 IP 地址。目前，由于缺少网络[路由](https://cloud.google.com/compute/docs/vpc/routes)，不同节点上的 Pod 之间无法互相通信。

在本实验中，你将为每个 worker 节点创建一条路由，将节点的 Pod CIDR 范围映射到该节点的内部 IP 地址。

> 还有[其他方式](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this)可以实现 Kubernetes 网络模型。

## 路由表

在本节中，你将收集创建路由所需的信息。

打印每个 worker 实例的内部 IP 地址和 Pod CIDR 范围：

```bash
{
  SERVER_IP=$(grep server machines.txt | cut -d " " -f 1)
  NODE_0_IP=$(grep node-0 machines.txt | cut -d " " -f 1)
  NODE_0_SUBNET=$(grep node-0 machines.txt | cut -d " " -f 4)
  NODE_1_IP=$(grep node-1 machines.txt | cut -d " " -f 1)
  NODE_1_SUBNET=$(grep node-1 machines.txt | cut -d " " -f 4)
}
```

```bash
ssh root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```

```bash
ssh root@node-0 <<EOF
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF
```

```bash
ssh root@node-1 <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
EOF
```

## 验证

```bash
ssh root@server ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160 
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

```bash
ssh root@node-0 ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

```bash
ssh root@node-1 ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

下一步: [冒烟测试](lab12-smoke-test.md)
