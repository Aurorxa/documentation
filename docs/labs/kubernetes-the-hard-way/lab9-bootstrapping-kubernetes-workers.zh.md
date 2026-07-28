---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova, Franco Colussi
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 9: 引导 Kubernetes Worker 节点

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将引导两个 Kubernetes worker 节点。以下组件将安装在 `node-0` 和 `node-1` 上：[runc](https://github.com/opencontainers/runc)、[cni 网络插件](https://github.com/containernetworking/cni)、[containerd](https://github.com/containerd/containerd)、[kubelet](https://kubernetes.io/docs/admin/kubelet) 和 [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies)。

## 前提条件

本实验中的命令必须同时在每个 worker 实例上运行：`node-0` 和 `node-1`。使用 `ssh` 命令通过 `jumpbox` 登录到每个 worker 实例。

```bash
ssh root@node-0
```

和

```bash
ssh root@node-1
```

## 搭建单个 Worker 节点

> 在 `node-0` 和 `node-1` 上运行以下步骤。

将 /etc/hosts 条目添加到本实验中的本地机器：

```bash
{
cat >> /etc/hosts << EOF
  XXX.XXX.XXX.XXX server.kubernetes.local server
  XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
  XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
EOF
}
```

### 安装 containerd

从 `jumpbox` 获取 `containerd` 二进制文件：

```bash
scp root@jumpbox:~/kubernetes-the-hard-way/downloads/containerd-2.0.3-linux-amd64.tar.gz .
```

提取 `containerd` 二进制文件并安装到 `/usr`：

```bash
tar -xvf containerd-2.0.3-linux-amd64.tar.gz
mv bin/* /usr/local/bin/
```

下载并配置 `containerd` 服务和 socket：

```bash
{
mkdir -p /usr/local/lib/systemd/system/
scp root@jumpbox:~/kubernetes-the-hard-way/containerd.service /usr/local/lib/systemd/system/containerd.service
scp root@jumpbox:~/kubernetes-the-hard-way/containerd.socket /usr/local/lib/systemd/system/containerd.socket
}
```

```bash
systemctl daemon-reload
systemctl enable --now containerd
```

### 安装 runc

从 `jumpbox` SSH 获取 `runc` 二进制文件：

```bash
scp root@jumpbox:~/kubernetes-the-hard-way/downloads/runc.amd64 .
```

安装 `runc`：

```bash
{
install -m 755 runc.amd64 /usr/local/sbin/runc
}
```

### 安装 CNI 插件

从 `jumpbox` 获取 `cni` 插件：

```bash
scp root@jumpbox:~/kubernetes-the-hard-way/downloads/cni-plugins-linux-amd64-v1.6.2.tgz .
```

安装 `cni` 插件：

```bash
mkdir -p /opt/cni/bin
tar -xvf cni-plugins-linux-amd64-v1.6.2.tgz -C /opt/cni/bin/
```

检查 `containerd` 状态：

```bash
systemctl status containerd
```

### 配置 Kubelet

安装 `crictl`：

```bash
{
scp root@jumpbox:~/kubernetes-the-hard-way/Downloads/crictl-v1.32.0-linux-amd64.tar.gz .
tar -xvf crictl-v1.32.0-linux-amd64.tar.gz
cp crictl /usr/local/bin/
}
```

配置 `crictl`：

```bash
cat <<EOF | tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false
EOF
```

```bash
HOSTNAME=$(hostname)
```

```bash
mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
mv ca.pem /var/lib/kubernetes/
```

> 上述命令假设在之前的实验中，已使用 SSH 将密钥、证书和 kubeconfig 文件传送到各 worker 实例。

创建 `kubelet-config.yaml` 配置文件：

```bash
cat <<EOF | tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/etc/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

> `resolvConf` 配置用于避免使用 `systemd-resolved` 运行 `kubelet` 时出现循环，尤其是在带有 `resolvconf` 的 Ubuntu 系统上，`/etc/resolv.conf` 指向 127.0.0.53。对于不同的 Linux 发行版，情况可能有所不同，并非所有情况都适用。

创建 `kubelet.service` systemd 单元文件：

```bash
cat <<EOF | tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime-endpoint=unix:///run/containerd/containerd.sock \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### 配置 Kubernetes Proxy

```bash
cp kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

创建 `kube-proxy-config.yaml` 配置文件：

```bash
cat <<EOF | tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```

创建 `kube-proxy.service` systemd 单元文件：

```bash
cat <<EOF | tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### 下载 Worker 二进制文件并启动 Worker 服务

从 `jumpbox` 获取 kubelet、kube-proxy 和 kubectl：

```bash
scp root@jumpbox:~/kubernetes-the-hard-way/downloads/kubelet .
scp root@jumpbox:~/kubernetes-the-hard-way/downloads/kube-proxy .
```

```bash
  mkdir -p \
    /var/lib/kubelet \
    /var/lib/kube-proxy \
    /var/lib/kubernetes/pki \
    /var/run/kubernetes
```

```bash
  chmod +x kubelet kube-proxy

  cp kubelet kube-proxy /usr/local/bin/
```

```bash
  systemctl daemon-reload

  systemctl enable kubelet kube-proxy

  systemctl start kubelet kube-proxy
```

## 验证

从 `jumpbox` 机器执行验证：

```bash
ssh root@server \
    "kubectl get nodes \
    --kubeconfig admin.kubeconfig"
```

```text
NAME     STATUS   ROLES    AGE     VERSION
node-0   Ready    <none>   1m      v1.32.0
node-1   Ready    <none>   1m      v1.32.0
```

下一步: [配置 kubectl 远程访问](lab10-configuring-kubectl.md)
