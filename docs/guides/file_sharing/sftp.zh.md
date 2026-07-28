---
title: 安全 SFTP 服务器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - sftp
  - 安全
  - ssh
---

# 安全 SFTP 服务器 —— 带 SSH 隔离

## 引言

SFTP（SSH File Transfer Protocol，SSH 文件传输协议）是一种安全的文件传输协议，它利用 SSH 提供加密通道来传输文件。与传统的 FTP 相比，SFTP 提供了更强的安全性，所有数据（包括认证信息）都会被加密。此外，SFTP 通过 SSH 使用单一端口，简化了防火墙配置，并且可以使用 SSH 密钥进行身份验证。

通过将 SFTP 用户隔离(chroot)到其主目录，您可以限制用户只能在其指定目录内进行文件操作，而无法访问其他系统目录，从而增强安全性。

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验

## 创建隔离环境

首先，创建一个专用于 SFTP 的组：

```bash
groupadd sftpusers
```

创建一个专用用户用于此演示：

```bash
useradd -g sftpusers -d /upload -s /sbin/nologin sftp_user
passwd sftp_user
```

现在，创建目录结构并修改权限。隔离(chroot)功能要求主目录上的权限非常严格。

```bash
mkdir -p /sftp/sftp_user/upload
chown root:root /sftp/sftp_user
chown sftp_user:sftpusers /sftp/sftp_user/upload
chmod 755 /sftp/sftp_user
```

## 配置 SSH

编辑 `/etc/ssh/sshd_config`，添加或找到 `Subsystem sftp` 行，并将其替换为：

```text
Subsystem sftp internal-sftp
```

在文件末尾，添加以下内容来创建隔离配置：

```text
Match Group sftpusers
    ChrootDirectory /sftp/%u
    ForceCommand internal-sftp
    PasswordAuthentication yes
    PermitTunnel no
    AllowAgentForwarding no
    AllowTcpForwarding no
    X11Forwarding no
```

测试 SSH 配置：

```bash
sshd -t
```

如果没有错误，重启 SSH 服务：

```bash
systemctl restart sshd
```

## 配置防火墙

确认防火墙允许 SSH 流量：

```bash
firewall-cmd --add-service=ssh --permanent
firewall-cmd --reload
```

## 测试

```bash
sftp sftp_user@192.168.0.10
```

登录后，进入 `upload` 目录：

```bash
cd upload
```

现在您只能在此受限环境中进行文件操作。

您还可以使用图形化 SFTP 客户端，如 FileZilla、WinSCP（Windows）或 Nautilus（Linux），这些客户端都支持通过 SFTP 连接。

## 结论

SFTP 通过 SSH 提供了一个安全的文件传输解决方案。通过结合 SSH 隔离的最佳实践，您可以创建一个受限的文件共享环境，以保护系统免遭未经授权的访问。这种方法比传统的 FTP 更安全，且更易于防火墙配置。
