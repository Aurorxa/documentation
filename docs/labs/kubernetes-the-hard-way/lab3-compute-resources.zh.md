---
author: Wale Soyinka 
contributors: Steven Spencer
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 3: 配置计算资源

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

Kubernetes 需要一组机器来托管 Kubernetes 控制平面以及最终运行容器的 worker 节点。在本实验中，你将提供搭建 Kubernetes 集群所需的机器。

## 机器数据库

本教程将利用一个文本文件作为机器数据库，用于存储在搭建 Kubernetes 控制平面和 worker 节点时将使用的各种机器属性。以下模式表示机器数据库中的条目，每个条目占一行：

```text
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
```

每列对应机器的 IP 地址 `IPV4_ADDRESS`、完全限定域名 `FQDN`、主机名 `HOSTNAME` 和 IP 子网 `POD_SUBNET`。Kubernetes 为每个 `pod` 分配一个 IP 地址，而 `POD_SUBNET` 表示集群中每台机器为此目的分配的唯一切片 IP 地址范围。

以下是类似于创建本教程所使用的机器数据库示例。请注意隐去的 IP 地址。你可以为机器分配任何 IP 地址，只要它们彼此之间以及 `jumpbox` 可以互相访问即可。

```bash
cat machines.txt
```

```text
XXX.XXX.XXX.XXX server.kubernetes.local server  
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0 10.200.0.0/24
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1 10.200.1.0/24
```

现在轮到你了，创建一个 `machines.txt` 文件，包含你将用于创建 Kubernetes 集群的三台机器的详细信息。你可以使用上面的示例机器数据库来添加机器的详细信息。

## 配置 SSH 访问

你将使用 SSH 配置集群中的机器。验证你对机器数据库中列出的每台机器拥有 `root` SSH 访问权限。你可能需要通过更新 `sshd_config` 文件并重启 SSH 服务器来启用每台节点上的 root SSH 访问。

### 启用 root SSH 访问

如果你已经拥有每台机器的 `root` SSH 访问权限，可以跳过本节。

Rocky Linux 的新安装默认禁用 `root` 用户的 SSH 访问。这是出于安全原因，因为 `root` 用户对类 Unix 系统拥有完全的管理控制权。弱密码对于连接到互联网的机器来说是很糟糕的。如前所述，你将启用 `root` SSH 访问以简化本教程中的步骤。安全性是一种取舍；在这种情况下，你正在为便利性进行优化。

使用 SSH 和你的用户账户登录到每台机器，然后使用 `su` 命令切换到 `root` 用户：

```bash
su - root
```

编辑 `/etc/ssh/sshd_config` SSH 守护进程配置文件，将 `PermitRootLogin` 选项设置为 `yes`：

```bash
sed -i \
  's/^#PermitRootLogin.*/PermitRootLogin yes/' \
  /etc/ssh/sshd_config
```

重启 `sshd` SSH 服务器以应用更新后的配置文件：

```bash
systemctl restart sshd
```

### 生成和分发 SSH 密钥

在这里，你将生成一个 SSH 密钥对并将其分发到 `server`、`node-0` 和 `node-1` 机器，你将在整个教程中使用它们在这些机器上运行命令。请从 `jumpbox` 机器运行以下命令。

生成一个新的 SSH 密钥：

```bash
ssh-keygen
```

在此处的提示中按 ++enter++ 接受所有默认值：

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

将 SSH 公钥复制到每台机器：

```bash
while read IP FQDN HOST SUBNET; do 
  ssh-copy-id root@${IP}
done < machines.txt
```

添加每个密钥后，验证 SSH 公钥访问是否正常工作：

```bash
while read IP FQDN HOST SUBNET; do 
  ssh -n root@${IP} uname -o -m
done < machines.txt
```

```text
x86_64 GNU/Linux
x86_64 GNU/Linux
x86_64 GNU/Linux
```

## 主机名

在本节中，你将为主机名分配给 `server`、`node-0` 和 `node-1` 机器。从 `jumpbox` 向每台机器执行命令时将使用主机名。主机名在集群内部也扮演着重要角色。Kubernetes 客户端将使用 `server` 主机名向 Kubernetes API 服务器发送命令，而不是使用 IP 地址。每个 worker 机器 `node-0` 和 `node-1` 在注册到给定 Kubernetes 集群时也会使用主机名。

要为每台机器配置主机名，请在 `jumpbox` 上运行以下命令。

在 `machines.txt` 文件中列出的每台机器上设置主机名：

```bash
while read IP FQDN HOST SUBNET; do
    ssh -n root@${IP} cp /etc/hosts /etc/hosts.bak 
    CMD="sed -i 's/^127.0.0.1.*/127.0.0.1\t${FQDN} ${HOST}/' /etc/hosts"
    ssh -n root@${IP} "$CMD"
    ssh -n root@${IP} hostnamectl hostname ${HOST}
done < machines.txt
```

验证每台机器上设置的主机名：

```bash
while read IP FQDN HOST SUBNET; do
  ssh -n root@${IP} hostname --fqdn
done < machines.txt
```

```text
server.kubernetes.local
node-0.kubernetes.local
node-1.kubernetes.local
```

## 主机查找表

在本节中，你将生成一个 `hosts` 文件，并将其附加到 `jumpbox` 的 `/etc/hosts` 文件以及本教程中使用的所有三台集群成员的 `/etc/hosts` 文件中。这将使每台机器都能够使用主机名（如 `server`、`node-0` 或 `node-1`）进行访问。

创建一个新的 `hosts` 文件并添加一个头部以标识正在添加的机器：

```bash
echo "" > hosts
echo "# Kubernetes The Hard Way" >> hosts
```

为 `machines.txt` 文件中的每台机器生成一个主机条目并追加到 `hosts` 文件中：

```bash
while read IP FQDN HOST SUBNET; do 
    ENTRY="${IP} ${FQDN} ${HOST}"
    echo $ENTRY >> hosts
done < machines.txt
```

查看 `hosts` 文件中的主机条目：

```bash
cat hosts
```

```text

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

## 将 `/etc/hosts` 条目添加到本地机器

在本节中，你将把 `hosts` 文件中的 DNS 条目追加到 `jumpbox` 机器上的本地 `/etc/hosts` 文件中。

将 `hosts` 中的 DNS 条目追加到 `/etc/hosts`：

```bash
cat hosts >> /etc/hosts
```

验证 `/etc/hosts` 文件的更新：

```bash
cat /etc/hosts
```

```text
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
```

你现在应该能够使用主机名 SSH 连接到 `machines.txt` 文件中列出的每台机器。

```bash
for host in server node-0 node-1
   do ssh root@${host} uname -o -m -n
done
```

```text
server x86_64 GNU/Linux
node-0 x86_64 GNU/Linux
node-1 x86_64 GNU/Linux
```

## 将 `/etc/hosts` 条目添加到远程机器

在本节中，你将把 `hosts` 中的主机条目追加到 `machines.txt` 文本文件中列出的每台机器的 `/etc/hosts`。

将 `hosts` 文件复制到每台机器并追加内容到 `/etc/hosts`：

```bash
while read IP FQDN HOST SUBNET; do
  scp hosts root@${HOST}:~/
  ssh -n \
    root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
```

你现在可以从 `jumpbox` 机器或 Kubernetes 集群中三台机器中的任意一台使用主机名连接机器。你现在可以使用 `server`、`node-0` 或 `node-1` 这样的主机名而不是 IP 地址来连接机器。

下一步: [配置 CA 并生成 TLS 证书](lab4-certificate-authority.md)
