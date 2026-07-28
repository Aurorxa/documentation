---
title: 安全 FTP 服务器 —— vsftpd
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - ftp
  - 安全
  - vsftpd
---

# 安全 FTP 服务器 —— vsftpd

## 引言

`vsftpd` 是 "Very Secure FTP Daemon" 的缩写，它是一个专为 Unix 和 Linux 系统设计的、轻量级且安全的 FTP 文件服务器。`vsftpd` 是在 GPL 许可证下发布的 FTP 服务器软件，以其速度、稳定性和安全性闻名。

本指南将介绍如何在 Rocky Linux 上安装和配置 `vsftpd`，重点是安全配置。

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验

## 安装 `vsftpd`

```bash
dnf install vsftpd
```

## 配置 `vsftpd`

备份默认配置文件：

```bash
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.orig
```

编辑 `/etc/vsftpd/vsftpd.conf`：

```text
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
listen=YES
listen_ipv6=NO
pam_service_name=vsftpd
userlist_enable=YES
userlist_deny=NO
userlist_file=/etc/vsftpd/user_list
tcp_wrappers=YES
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
rsa_cert_file=/etc/pki/tls/certs/vsftpd.crt
rsa_private_key_file=/etc/pki/tls/private/vsftpd.key
```

### TLS 证书

为 vsftpd 创建自签名证书或使用现有的证书：

```bash
mkdir -p /etc/pki/tls/certs /etc/pki/tls/private
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/pki/tls/private/vsftpd.key -out /etc/pki/tls/certs/vsftpd.crt
```

然后重新设置文件权限：

```bash
chmod 600 /etc/pki/tls/private/vsftpd.key
chmod 600 /etc/pki/tls/certs/vsftpd.crt
```

### 创建用户列表

将允许访问 FTP 服务器的用户添加到 `/etc/vsftpd/user_list` 文件中：

```bash
echo "username" >> /etc/vsftpd/user_list
```

## 配置防火墙

使用 `firewalld` 允许 FTP：

```bash
firewall-cmd --add-service=ftp --permanent
firewall-cmd --reload
```

如果需要被动模式端口，请添加以下规则：

```bash
firewall-cmd --add-port=40000-40100/tcp --permanent
firewall-cmd --reload
```

启动并启用 vsftpd 服务：

```bash
systemctl enable --now vsftpd
```

## 测试

使用 `ftp` 客户端测试连接：

```bash
ftp localhost
```

或者使用 `lftp`（推荐）：

```bash
lftp -u username localhost
```

## 结论

`vsftpd` 是一个轻量级但安全的 FTP 服务器，适用于简单的文件共享。通过适当的配置，包括 TLS 加密和 chroot 限制，您可以确保文件传输的安全性和私密性。
