---
author: Wale Soyinka 
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 2: 搭建 Jumpbox

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将把四台机器中的一台设置为 `jumpbox`。你将使用这台机器来运行本教程中的命令。虽然使用专用机器是为了确保一致性，但你也可以从几乎任何机器上运行这些命令，包括运行 macOS 或 Linux 的个人工作站。

可以将 `jumpbox` 视为你在从头搭建 Kubernetes 集群时用作"大本营"的管理机器。在正式开始之前，你需要安装一些命令行工具，并克隆 Kubernetes The Hard Way 的 git 仓库，其中包含一些额外的配置文件，你将在整个教程中使用它们来配置各种 Kubernetes 组件。

登录到 `jumpbox`：

```bash
ssh root@jumpbox
```

为了方便起见，你将使用 `root` 用户运行所有命令，这有助于减少执行配置所需的命令数量。

## 安装命令行工具

以 `root` 用户登录 `jumpbox` 机器后，安装将在教程中用于执行各种任务的命令行工具：

```bash
sudo dnf -y install wget curl vim openssl git
```

## 同步 GitHub 仓库

现在是时候下载本教程的副本，其中包含你将从零构建 Kubernetes 集群所需的配置文件和使用。使用 `git` 命令克隆 Kubernetes The Hard Way git 仓库：

```bash
git clone --depth 1 \
  https://github.com/wsoyinka/kubernetes-the-hard-way.git
```

切换到 `kubernetes-the-hard-way` 目录：

```bash
cd kubernetes-the-hard-way
```

这将是本教程后续部分的工作目录。如果你迷路了，运行 `pwd` 命令验证你在 `jumpbox` 上运行命令时是否处于正确的目录：

```bash
pwd
```

```text
/root/kubernetes-the-hard-way
```

## 下载二进制文件

在这里，你将下载各种 Kubernetes 组件的二进制文件。将这些二进制文件存储在 `jumpbox` 上的 `Downloads` 目录中。这将减少完成本教程所需的互联网带宽，因为你无需为 Kubernetes 集群中的每台机器重复下载这些二进制文件。

`download.txt` 文件列出了你需要下载的二进制文件，你可以使用 `cat` 命令查看：

```bash
cat downloads.txt
```

使用 `wget` 命令将 `downloads.txt` 文件中列出的二进制文件下载到名为 `downloads` 的目录中：

```bash
wget -q --show-progress \
  --https-only \
  --timestamping \
  -P downloads \
  -i downloads.txt
```

根据你的互联网连接速度，下载 `584` 兆字节的二进制文件可能需要一些时间。下载完成后，你可以使用 `ls` 命令列出它们：

```bash
ls -loh downloads
```

```text
total 557M
-rw-r--r--. 1 root 51M Jan  6 11:13 cni-plugins-linux-amd64-v1.6.2.tgz
-rw-r--r--. 1 root 36M Feb 28 14:09 containerd-2.0.3-linux-amd64.tar.gz
-rw-r--r--. 1 root 19M Dec  9 04:16 crictl-v1.32.0-linux-amd64.tar.gz
-rw-r--r--. 1 root 17M Feb 25 14:19 etcd-v3.4.36-linux-amd64.tar.gz
-rw-r--r--. 1 root 89M Dec 11 16:12 kube-apiserver
-rw-r--r--. 1 root 82M Dec 11 16:12 kube-controller-manager
-rw-r--r--. 1 root 55M Dec 11 16:12 kubectl
-rw-r--r--. 1 root 74M Dec 11 16:12 kubelet
-rw-r--r--. 1 root 64M Dec 11 16:12 kube-proxy
-rw-r--r--. 1 root 63M Dec 11 16:12 kube-scheduler
-rw-r--r--. 1 root 11M Feb 13 20:19 runc.amd64
```

## 安装 `kubectl`

在本节中，你将在 `jumpbox` 机器上安装 `kubectl`，即 Kubernetes 官方的客户端命令行工具。在本教程后续部分完成集群配置后，你将使用 `kubectl` 与 Kubernetes 控制平面进行交互。

使用 `chmod` 命令使 `kubectl` 二进制文件可执行，并将其移动到 `/usr/local/bin/` 目录：

```bash
  chmod +x downloads/kubectl
  cp downloads/kubectl /usr/local/bin/
```

由于 `kubectl` 安装已完成，你可以通过运行 `kubectl` 命令来验证：

```bash
kubectl version --client
```

```text
Client Version: v1.32.0
Kustomize Version: v5.5.0
```

至此，你已搭建了一个 `jumpbox`，其中包含完成本教程实验所需的所有命令行工具和实用程序。

下一步: [配置计算资源](lab3-compute-resources.md)
