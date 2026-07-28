---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 8: 引导 Kubernetes 控制平面

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将在 `server` 实例上引导 Kubernetes 控制平面。以下组件将被安装：API Server、Scheduler 和 Controller Manager。

## 前提条件

请从 `jumpbox` 将控制平面二进制文件复制到 `server` 实例：

```bash
scp \
  downloads/kube-apiserver \
  downloads/kube-controller-manager \
  downloads/kube-scheduler \
  downloads/kubectl \
  root@server:~/
```

本实验中的命令可能影响整个集群，并且只在 `server` 实例上运行。使用 `ssh` 命令登录到 `server` 实例：

```bash
ssh root@server
```

## 配置 Kubernetes API Server

```bash
mkdir -p /var/lib/kubernetes/

mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem \
    /var/lib/kubernetes/
```

Kubernetes API 服务器将在集群内部 IP 地址上运行。将 `INTERNAL_IP` 环境变量设置为 `server` 实例的 IP 地址：

```bash
INTERNAL_IP=$(grep server machines.txt | cut -d " " -f 1)
```

使用 `INTERNAL_IP` 环境变量创建 API Server systemd 单元文件：

```bash
cat <<EOF | tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NodeRestriction,ServiceAccount \\
  --enable-bootstrap-token-auth=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://${INTERNAL_IP}:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --runtime-config=api/all=true \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-account-signing-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-account-issuer=https://${INTERNAL_IP}:6443 \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## 配置 Kubernetes Controller Manager

将 `kube-controller-manager` kubeconfig 移动到正确的位置：

```bash
cp kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

创建 `kube-controller-manager` systemd 单元文件：

```bash
cat <<EOF | tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --allocate-node-cidrs=true \\
  --authentication-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authorization-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --bind-address=127.0.0.1 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --node-cidr-mask-size=24 \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## 配置 Kubernetes Scheduler

将 `kube-scheduler` kubeconfig 移动到正确的位置：

```bash
cp kube-scheduler.kubeconfig /var/lib/kubernetes/
```

创建 `kube-scheduler` systemd 单元文件：

```bash
cat <<EOF | tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --authentication-kubeconfig=/var/lib/kubernetes/kube-scheduler.kubeconfig \\
  --authorization-kubeconfig=/var/lib/kubernetes/kube-scheduler.kubeconfig \\
  --bind-address=127.0.0.1 \\
  --kubeconfig=/var/lib/kubernetes/kube-scheduler.kubeconfig \\
  --leader-elect=true
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### 启动 Controller 服务

```bash
  chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl

  cp kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```

```bash
  systemctl daemon-reload

  systemctl enable kube-apiserver kube-controller-manager kube-scheduler

  systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

## 验证

```bash
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

```text
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE   ERROR
scheduler            Healthy   ok        
controller-manager   Healthy   ok        
etcd-0               Healthy   ok
```

```text
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```

这将列出调度器（scheduler）、控制器管理器（controller manager）和 *etcd-0* 服务器，状态为 `Healthy`。

!!! note

    使用 `kubectl` 测试运行正常后，请记得删除 `admin.kubeconfig` 文件，以防止被非特权用户在 `server` 实例上直接运行。

```bash
rm admin.kubeconfig
```

## RBAC for Kubelet Authorization（Kubelet 授权的 RBAC）

在本节中，你将配置 RBAC 权限，允许 Kubernetes API Server 访问每个 worker 节点上的 Kubelet API。访问 Kubelet API 对于检索指标、日志以及在 pod 中执行命令是必需的。

> 本教程设置 Kubelet `--authorization-mode` 标志为 `Webhook`。Webhook 模式使用 [SubjectAccessReview](https://kubernetes.io/docs/admin/authorization/#checking-api-access) API 来确定授权。

```bash
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

```bash
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

```bash
kubectl cluster-info --kubeconfig admin.kubeconfig
```

```text
Kubernetes control plane is running at https://127.0.0.1:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

下一步: [引导 Kubernetes Worker 节点](lab9-bootstrapping-kubernetes-workers.md)
