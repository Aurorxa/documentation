---
title: SSH 证书颁发机构与密钥签名
author: Julian Patocki
contributors: Steven Spencer, Ganna Zhyrnova
tags:
    - security
    - ssh
    - keygen
    - certificates
---

## 前提条件

* 能够使用命令行工具
* 从命令行管理内容
* 之前有 SSH 密钥生成经验有帮助，但不是必需的
* 对 SSH 和公钥基础设施的基本理解有帮助但非必需
* 一个运行 sshd 守护进程的服务器。

## 简介

如果无法验证远程主机密钥的指纹，初始 SSH 连接与远程主机是不安全的。使用 CA（Certificate Authority，证书颁发机构）签远程主机的公钥使得初始连接对信任该 CA 的每个用户都是安全的。

CA 还可以用于签用户 SSH 密钥。无需将密钥分发到每个远程主机，一个签名就足以授权用户登录到多个服务器。

## 目标

* 提高 SSH 连接的安全性。
* 改进入职流程和密钥管理。

## 说明

* Vim 是作者首选的文本编辑器。使用其他文本编辑器，如 nano 或其他，也完全可接受。
* 使用 `sudo` 或 `root` 意味着提升的权限。

## 初始连接

要保护初始连接，您需要事先了解密钥指纹。您可以优化并将此部署过程集成到新主机中。

在远程主机上显示密钥指纹：

```bash
user@rocky-vm ~]$ ssh-keygen -E sha256 -l -f /etc/ssh/ssh_host_ed25519_key.pub 
256 SHA256:bXWRZCpppNWxXs8o1MyqFlmfO8aSG+nlgJrBM4j4+gE no comment (ED25519)
```

从客户端建立初始 SSH 连接。密钥指纹被显示，可以与之前获取的进行比较：

```bash
[user@rocky ~]$ ssh user@rocky-vm.example.com
The authenticity of host 'rocky-vm.example (192.168.56.101)' can't be established.
ED25519 key fingerprint is SHA256:bXWRZCpppNWxXs8o1MyqFlmfO8aSG+nlgJrBM4j4+gE.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

## 创建证书颁发机构

创建一个 CA（私钥和公钥）并将公钥放入客户端的 `known_hosts` 文件中，以识别属于 example.com 域的所有主机：

```bash
[user@rocky ~]$ ssh-keygen -b 4096 -t ed25519 -f CA
[user@rocky ~]$ echo '@cert-authority *.example.com' $(cat CA.pub) >> ~/.ssh/known_hosts
```

其中：

* **-b**：密钥长度，以字节为单位
* **-t**：密钥类型：rsa、ed25519、ecdsa...
* **-f**：输出密钥文件

或者，您可以通过编辑 SSH 配置文件 `/etc/ssh/ssh_config` 在系统范围内指定 `known_hosts` 文件：

```bash
Host *
    GlobalKnownHostsFile /etc/ssh/ssh_known_hosts
```

## 签公钥

创建用户 SSH 密钥并签名：

```bash
[user@rocky ~]$ ssh-keygen -b 4096 -t ed2119
[user@rocky ~]$ ssh-keygen -s CA -I user -n user -V +55w  id_ed25519.pub
```

通过 `scp` 获取服务器的公钥并签名：

```bash
[user@rocky ~]$ scp user@rocky-vm.example.com:/etc/ssh/ssh_host_ed25519_key.pub .
[user@rocky ~]$ ssh-keygen -s CA -I rocky-vm -n rocky-vm.example.com -h -V +55w ssh_host_ed25519_key.pub
```

其中：

* **-s**：签名密钥
* **-I**：标识证书用于日志记录的名称
* **-n**：标识与证书关联的名称（主机或用户，一个或多个）（如果未指定，证书对所有用户或主机有效）
* **-h**：将证书定义为主机密钥，而不是客户端密钥
* **-V**：证书的有效期

## 建立信任

复制远程主机的证书，以便远程主机在被连接时与其公钥一起呈现：

```bash
[user@rocky ~]$ scp ssh_host_ed25519_key-cert.pub root@rocky-vm.example.com:/etc/ssh/
```

将 CA 的公钥复制到远程主机，使其信任由 CA 签的证书：

```bash
[user@rocky ~]$ scp CA.pub root@rocky-vm.example.com:/etc/ssh/
```

将以下行添加到 `/etc/ssh/sshd_config` 文件中，以指定服务器使用之前复制的密钥和证书，并信任 CA 来识别用户：

```bash
[user@rocky ~]$ ssh user@rocky-vm.example.com
[user@rocky-vm ~]$ sudo vim /etc/ssh/sshd_config
```

```bash
HostKey /etc/ssh/ssh_host_ed25519_key
HostCertificate /etc/ssh/ssh_host_ed25519_key-cert.pub
TrustedUserCAKeys /etc/ssh/CA.pub
```

在服务器上重启 `sshd` 服务：

```bash
[user@rocky-vm ~]$ systemctl restart sshd
```

## 测试连接

从您的 `known_hosts` 文件中移除远程服务器的指纹，并通过建立 SSH 连接来验证设置：

```bash
[user@rocky ~]$ ssh-keygen -R rocky-vm.example.com
[user@rocky ~]$ ssh user@rocky-vm.example.com
```

## 密钥吊销

吊销主机或用户密钥可能对整体环境的安全至关重要。因此，存储之前签的公钥，以便以后可以吊销它们是很重要的。

创建一个空的吊销列表并吊销 user2 的公钥：

```bash
[user@rocky ~]$ sudo ssh-keygen -k -f /etc/ssh/revokation_list.krl
[user@rocky ~]$ sudo ssh-keygen -k -f /etc/ssh/revokation_list.krl -u /path/to/user2_id_ed25519.pub
```

将吊销列表复制到远程主机并在 `sshd_config` 文件中指定它：

```bash
[user@rocky ~]$ scp /etc/ssh/revokation_list.krl root@rocky-vm.example.com:/etc/ssh/
[user@rocky ~]$ ssh user@rocky-vm.example.com
[user@rocky ~]$ sudo vim /etc/ssh/sshd_config
```

以下行指定了吊销列表：

```bash
RevokedKeys /etc/ssh/revokation_list.krl
```

需要重启 SSHD 守护进程以使配置重新加载：

```bash
[user@rocky-vm ~]$ sudo systemctl restart sshd
```

User2 被服务器拒绝：

```bash
[user2@rocky ~]$ ssh user2@rocky-vm.example.com
user2@rocky-vm.example.com: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

也可以吊销服务器密钥：

```bash
[user@rocky ~]$ sudo ssh-keygen -k -f /etc/ssh/revokation_list.krl -u /path/to/ssh_host_ed25519_key.pub
```

`/etc/ssh/ssh_config` 中的以下行在系统范围内应用主机吊销列表：

```bash
Host *
        RevokedHostKeys /etc/ssh/revokation_list.krl
```

尝试连接到该主机的结果如下：

```bash
[user@rocky ~]$ ssh user@rocky-vm.example.com
Host key ED25519-CERT SHA256:bXWRZCpppNWxXs8o1MyqFlmfO8aSG+nlgJrBM4j4+gE revoked by file /etc/ssh/revokation_list.krl
```

维护和更新吊销列表非常重要。您可以自动化该过程以确保所有主机和用户可以访问最新的吊销列表。

## 结论

SSH 是管理远程服务器最有价值的协议之一。实现证书颁发机构可能有帮助，特别是在有许多服务器和用户的较大环境中。
维护吊销列表很重要。它可以在不替换整个关键基础设施的情况下轻松吊销被泄露的密钥。
