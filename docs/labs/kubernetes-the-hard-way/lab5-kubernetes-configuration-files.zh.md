---
author: Wale Soyinka
contributors: Steven Spencer
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 5: 生成用于认证的 Kubernetes 配置文件

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将生成 [Kubernetes 配置文件](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)，也称为 kubeconfig，它使 Kubernetes 客户端能够定位并认证到 Kubernetes API 服务器。

## 客户端认证配置文件

在本节中，你将生成 `kubelet`、`kube-controller-manager`、`kube-proxy`、`scheduler` 以及 `admin` 用户的 kubeconfig 文件。

创建完成后，你将把它们分发到相应的机器。

### Kubernetes API 服务器定位

每个 kubeconfig 需要一个要连接的 Kubernetes API 服务器。为支持高可用性，将使用分配给 Kubernetes API 服务器前端的外部负载均衡器的 IP 地址。在此，这些是 `machines.txt` 文件中 `server` 条目的详细信息。

确保打印 `kubernetes-the-hard-way` 的静态 IP 地址以供验证：

```bash
KUBERNETES_ADDRESS=$(grep server machines.txt | cut -d " " -f 1)
echo $KUBERNETES_ADDRESS
```

### kubelet Kubernetes 配置文件

为 `kubelet` 生成 kubeconfig 文件时，必须使用与 `kubelet` 的节点名称匹配的客户端证书。这将确保 Kubernetes [Node Authorizer](https://kubernetes.io/docs/reference/access-authn-authz/node/) 正确授权 `kubelet`。

> 以下命令需要在用于生成本教程中证书所在的同一目录中运行。如果不是，请参考并调整指向 `ca.pem` 和 `*-key.pem` 等文件的相对路径。

为每个 worker 节点生成 kubeconfig 文件：

```bash
for instance in node-0 node-1; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default \
    --kubeconfig=${instance}.kubeconfig
done
```

结果：

```text
node-0.kubeconfig
node-1.kubeconfig
```

### kube-proxy Kubernetes 配置文件

为 `kube-proxy` 服务生成 kubeconfig 文件：

```bash
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-proxy.kubeconfig
```

结果：

```text
kube-proxy.kubeconfig
```

### kube-controller-manager Kubernetes 配置文件

为 `kube-controller-manager` 服务生成 kubeconfig 文件：

```bash
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-controller-manager.kubeconfig
```

结果：

```text
kube-controller-manager.kubeconfig
```

### kube-scheduler Kubernetes 配置文件

为 `kube-scheduler` 服务生成 kubeconfig 文件：

```bash
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-scheduler.kubeconfig
```

结果：

```text
kube-scheduler.kubeconfig
```

### Admin Kubernetes 配置文件

为 `admin` 用户生成 kubeconfig 文件：

```bash
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default \
    --kubeconfig=admin.kubeconfig
```

结果：

```text
admin.kubeconfig
```

## 分发 Kubernetes 配置文件

将 `kubelet` 和 `kube-proxy` kubeconfig 文件复制到每个 worker 实例：

```bash
for instance in node-0 node-1; do
  scp ${instance}.kubeconfig kube-proxy.kubeconfig root@${instance}:~/
done
```

将 `admin`、`kube-controller-manager` 和 `kube-scheduler` kubeconfig 文件复制到 `server` 实例：

```bash
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig root@server:~/
```

下一步: [生成数据加密密钥](lab6-data-encryption-keys.md)
