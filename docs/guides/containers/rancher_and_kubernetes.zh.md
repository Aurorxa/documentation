---
title: 使用 Rancher 在 Rocky Linux 上安装 Kubernetes
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.6, 9.0
tags:
  - containers
  - kubernetes
  - docker
  - rancher
  - k3s
---

# 使用 Rancher 的 Kubernetes 集群安装

## 引言

本文档概述了在 Rocky Linux、CentOS、RHEL 或类似发行版上安装 [Rancher](https://www.rancher.com/) 的过程。Rancher 提供了一个基于 RKE（Rancher Kubernetes Engine，Rancher Kubernetes 引擎）和 K3S 的完整 Kubernetes 管理平台。它是经过认证的 Kubernetes 发行版，提供容器工作负载的管理。

由于 Kubernetes 是一个容器编排平台，在安装之前需要先安装一种容器技术。对于 RKE 和 K3S 安装方式，可以在 Docker Engine 和 [containerd](https://containerd.io/) 之间选择。

对于 RKE 安装，Docker 是目前唯一支持的容器运行时，而对于 K3S，Rancher 设计时允许根据需求任选其中之一。

在 Rocky Linux 8 上，`docker` 软件包可以通过官方仓库获取，而在 Rocky Linux 9 及更高版本上不再可用，但可以通过安装 `containerd` 来使用。我们的安装将使用 `containerd` 实现 K3S，而 Rancher 管理平台将在单独的 Docker 环境中运行。

这种配置的优势在于，Rancher 的管理平台运行在 Docker Engine 容器中，而 K3S 下游(downstream)集群则可以根据需要同时使用其中的一种，即在 Rocky Linux 8 上使用 Docker，Rocky Linux 9 上使用 `containerd`，从而形成一个灵活的安装方案。

!!! note "信息"

    本文使用 K3S 在单台机器上安装一个用于开发或测试的单节点集群。对于生产环境，需要有多个专用的服务器节点。

## 前提条件

* 一台运行 Rocky Linux 8 或 Rocky Linux 9 的服务器。
* 在没有 Root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）。
* 熟悉命令行操作。
* 具备使用 `vi`、`nano` 等命令行编辑器的经验。
* 对 Kubernetes 的使用有基本了解。

## 环境准备

安装 Rancher 的第一步是配置实验环境。Rancher 节点需要在其自身的 Docker Engine 内运行，并且我们希望它安装在与 K3S 集群相同的服务器上。我们必须创建一个与 Docker 运行分开的 K3S 环境。

所需的环境配置包括添加所需的软件仓库、安装容器工具，并确保正确启用环境模块(module)和仓库。

### Rocky Linux 9 配置

对于 Rocky Linux 9，我们需要添加 `containerd.io` 仓库，因为我们使用的是 `containerd` 容器运行时。

```bash
dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
dnf install containerd.io
```

### Rocky Linux 8 配置

对于 Rocky Linux 8，我们将使用 Docker 运行时，因此需要安装完整的 Docker Engine，同时还需要为 `--enablerepo` 选项启用所需的 Docker 仓库。

```bash
dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
dnf install -y docker-ce --enablerepo=docker-ce-stable
```

### 常用配置

两种系统都需要进行通用配置，包括启用 EPEL 和安装常用软件。

```bash
dnf install -y epel-release
dnf update -y
dnf install -y curl git jq nfs-utils
```

完成通用安装后，必须启用 `containerd` 并加载所需的模块(module)。

```bash
modprobe overlay
modprobe br_netfilter
cat <<EOF | tee /etc/modules-load.d/containerd.conf > /dev/null
overlay
br_netfilter
EOF
```

要满足内核设置的要求，需要准备 `/etc/sysctl.d/99-kubernetes-cri.conf` 文件：

```bash
cat <<EOF | tee /etc/sysctl.d/99-kubernetes-cri.conf > /dev/null
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

使用以下命令应用更改：

```bash
sysctl --system
```

### 为 `containerd` 启用 `cgroups v2`

Rancher 的 K3S 需要 `cgroups v2` 才能运行，因此我们必须确保 `containerd` 被配置为使用此版本。我们通过创建其默认配置文件来实现这一点：

```bash
containerd config default | tee /etc/containerd/config.toml > /dev/null
```

然后编辑 `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` 部分，并将 `SystemdCgroup` 设为 true。

```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  ...
  SystemdCgroup = true
```

然后启用 `containerd` 并设置开机自启：

```bash
systemctl enable --now containerd
```

### 禁用 SWAP

由于 kubelet 无法在启用 SWAP 的机器上工作，我们必须禁用 SWAP 并确保在 `/etc/fstab` 中将其注释掉。输入 `free -m` 命令查看当前状态。

```bash
swapoff -a
```

在 `/etc/fstab` 文件中，注释掉所有以 `swap` 开头的行，保存文件。

### 添加防火墙规则

对于单节点集群，为了便于测试，您可以简单地关闭防火墙：

```bash
systemctl disable --now firewalld
```

否则，请参考 Kubernetes 文档中的 [所需端口](https://kubernetes.io/docs/reference/networking/ports-and-protocols/) 表格，打开集群节点间通信所需的所有端口。

## Docker 安装（仅 Rocky Linux 8）

!!! note "注意"

    如果您正在设置 Rocky Linux 9，可以跳过此部分，因为 Rancher 服务器将在 Docker Engine 上运行，而 K3S 将使用 containerd。两者均可在 Rocky Linux 9 上共存。

    对于 Rocky Linux 8，Docker 需要作为 `root` 用户运行，因此我们也需要以 `root` 身份执行以下命令。

为 Docker 准备 `daemon.json` 配置文件。创建 `/etc/docker/daemon.json` 文件，内容如下：

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

在 `/etc/systemd/system/docker.service.d` 目录下创建一个包含 `docker.conf` 文件的目录：

```bash
mkdir -p /etc/systemd/system/docker.service.d
```

并添加 `http-proxy.conf` 文件，其中包含您的代理地址（如果适用）。

```bash
cat <<EOF | tee /etc/systemd/system/docker.service.d/http-proxy.conf > /dev/null
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=http://proxy.example.com:443"
Environment="NO_PROXY=localhost,127.0.0.1"
EOF
```

重启`docker` 服务：

```bash
systemctl daemon-reload
systemctl restart docker
```

启用 Docker（设置开机自启）并运行一个 `hello-world` 测试：

```bash
systemctl enable --now docker
docker run hello-world
```

## K3S Kubernetes 安装

现在所有的先决条件都满足了，您可以继续安装 K3S。K3S 安装命令必须作为 root 执行：

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -
```

通过显示节点和 pod 来验证安装是否成功：

```bash
kubectl get nodes
kubectl get pods --all-namespaces
```

此时，K3S 集群已安装并可以管理 Pod。如果一切正常，您现在可以安装 Rancher 管理平台以及所需的 `cert-manager`。

!!! warning "重要提示"

    如果没有 `cert-manager`，Rancher 管理平台将无法工作。在安装 Rancher 之前，您**必须**先安装它。

## `cert-manager` 安装

按照 Rancher 文档中的说明，配置 cert-manager 并安装。

从 [cert-manager 官方网站](https://cert-manager.io/docs/installation/kubectl/) 安装最新版本：

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/<VERSION>/cert-manager.crds.yaml
```

然后使用 `kubectl` 创建 `cert-manager` 所需的命名空间(namespace)：

```bash
kubectl create namespace cert-manager
```

添加 Jetstack 仓库：

```bash
helm repo add jetstack https://charts.jetstack.io
```

进行更新：

```bash
helm repo update
```

运行 cert-manager 的安装，使用 Helm 图表：

```bash
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version <VERSION>
```

分别将 `<VERSION>` 替换为实际的版本号。要检查 `cert-manager` 命名空间中的 pod，请运行：

```bash
kubectl get pods --namespace cert-manager
```

一切就绪后，您可以开始下一步，为 Rancher 管理平台创建命名空间并安装。

## Rancher 管理平台安装

接下来，使用 `kubectl` 为 Rancher 管理平台创建所需的命名空间：

```bash
kubectl create namespace cattle-system
```

现在使用 Helm 图表安装 Rancher 管理平台。安装时，您必须指定一个用于访问管理平台的主机名，并且必须执行以下命令。

!!! info "信息"

    如果您希望了解有关主机名选择和证书类型的更多信息，请参考 [Rancher 文档](https://ranchermanager.docs.rancher.com/v2.6/pages-for-subheaders/installation-and-upgrade-resources)。

```bash
helm install rancher rancher-latest/rancher \
   --namespace cattle-system \
   --set hostname=rancher.example.com \
   --set replicas=1
```

最后，验证 `cattle-system` 命名空间中的 Pod：

```bash
kubectl get pods --namespace cattle-system
```

## Rancher 仪表板访问

通过 Web 浏览器访问 Rancher 仪表板，地址为 `https://rancher.example.com`。它会要求您创建一个用户和密码完成设置。之后，您就可以使用 Rancher 管理您的 Kubernetes 集群了。

## 结论

本文涵盖了在 Rocky Linux 8 和 Rocky Linux 9 上安装 Rancher 管理平台及其 K3S 下游集群的步骤。Rancher 是一个强大的 Kubernetes 管理平台，提供了直观的 UI，帮助您轻松管理节点、工作负载和 Pod。
