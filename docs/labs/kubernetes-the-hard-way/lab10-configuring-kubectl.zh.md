---
author: Wale Soyinka
contributors: Steven Spencer
tags:
  - kubernetes
  - k8s
  - lab exercise
  - runc
  - containerd
  - etcd
  - kubectl
---

# 实验 10: 配置 `kubectl` 远程访问

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将基于 `admin` 用户凭据为 `kubectl` 命令行工具生成 kubeconfig 文件。

> 请从 `jumpbox` 机器运行本实验中的命令。

## Admin Kubernetes 配置文件

每个 kubeconfig 需要一个要连接的 Kubernetes API Server。

基于之前实验中的 `/etc/hosts` DNS 条目，你应该能够 ping 通 `server.kubernetes.local`。

```bash
curl -k --cacert ca.crt \
  https://server.kubernetes.local:6443/version
```

```text
{
  "major": "1",
  "minor": "32",
  "gitVersion": "v1.32.0",
  "gitCommit": "70d3cc986aa8221cd1dfb1121852688902d3bf53",
  "gitTreeState": "clean",
  "buildDate": "2024-12-11T17:59:15Z",
  "goVersion": "go1.23.3",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

生成适合以 `admin` 用户身份认证的 kubeconfig 文件：

```bash
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
```

运行上述命令后，结果将在 `kubectl` 命令行工具使用的默认位置 `~/.kube/config` 创建一个 kubeconfig 文件。这也意味着你可以直接运行 `kubectl` 命令而无需指定配置文件。

## 验证

检查远程 Kubernetes 集群的版本：

```bash
kubectl version
```

```text
Client Version: v1.32.0
Kustomize Version: v5.5.0
Server Version: v1.32.0
```

列出远程 Kubernetes 集群中的节点：

```bash
kubectl get nodes
```

```text
NAME     STATUS   ROLES    AGE   VERSION
node-0   Ready    <none>   30m   v1.31.2
node-1   Ready    <none>   35m   v1.31.2
```

下一步: [配置 Pod 网络路由](lab11-pod-network-routes.md)
