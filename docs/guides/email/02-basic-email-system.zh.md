---
title: 基础电子邮件系统
author: Steven Spencer
contributors: Ganna Zhyrnova, Colussi Franco
tested_with: 8.5, 8.6, 9.0
tags:
  - email
---

# 基础电子邮件系统设置

## 引言

本指南将带您了解如何使用 Postfix 作为 MTA（邮件传输代理）和 Dovecot 作为 MDA（邮件投递代理）设置一个基础的电子邮件系统。目的是提供一个功能简单的电子邮件服务器，但强调安全性——这是所有邮件服务器必须具备的。

!!! note

    实际生产环境配置通常还包括反垃圾邮件引擎、Webmail 接口等。但对于一个基础的收发邮件的服务器来说，Postfix + Dovecot 的配置就足够了。

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验
* 注册域名、公网 IP 以及正确配置的 DNS 记录

## 安装软件包

```bash
dnf install postfix dovecot
```

## Postfix 配置

备份默认配置文件：

```bash
cp /etc/postfix/main.cf /etc/postfix/main.cf.orig
```

编辑 `/etc/postfix/main.cf` 文件：

```text
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mynetworks = 127.0.0.0/8
home_mailbox = Maildir/
smtpd_banner = $myhostname ESMTP

# TLS 配置
smtpd_tls_cert_file = /etc/pki/tls/certs/mailserver.crt
smtpd_tls_key_file = /etc/pki/tls/private/mailserver.key
smtpd_use_tls = yes
smtpd_tls_auth_only = yes

# SASL 身份验证
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
```

### TLS 证书

如果还没有 TLS 证书，您可以创建一个自签名证书用于测试。对于生产环境，请使用 Let's Encrypt 或商业证书。

```bash
mkdir -p /etc/pki/tls/certs /etc/pki/tls/private
openssl req -new -x509 -days 3650 -nodes -out /etc/pki/tls/certs/mailserver.crt -keyout /etc/pki/tls/private/mailserver.key
```

## Dovecot 配置

备份默认配置文件：

```bash
cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.orig
```

编辑 `/etc/dovecot/dovecot.conf` 文件：

```text
protocols = imap pop3
listen = *

mail_location = maildir:~/Maildir
```

编辑 `/etc/dovecot/conf.d/10-auth.conf`，确保以下行未被注释：

```text
disable_plaintext_auth = yes
auth_mechanisms = plain login
```

编辑 `/etc/dovecot/conf.d/10-master.conf`，找到并修改 `service auth` 部分，使其包含：

```text
unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
}
```

编辑 `/etc/dovecot/conf.d/10-ssl.conf`：

```text
ssl = required
ssl_cert = </etc/pki/tls/certs/mailserver.crt
ssl_key = </etc/pki/tls/private/mailserver.key
```

## 启用并启动服务

```bash
systemctl enable --now postfix dovecot
```

## 配置防火墙

`firewalld`：

```bash
firewall-cmd --add-service=smtp --permanent
firewall-cmd --add-service=smtps --permanent
firewall-cmd --add-service=imap --permanent
firewall-cmd --add-service=imaps --permanent
firewall-cmd --add-service=pop3 --permanent
firewall-cmd --add-service=pop3s --permanent
firewall-cmd --reload
```

## 创建邮件用户

由于所有电子邮件用户都是 Linux 系统用户，因此可以通过标准命令进行添加：

```bash
useradd -m -s /sbin/nologin johndoe
passwd johndoe
```

## 测试

您可以使用 `telnet` 测试 SMTP：

```bash
telnet localhost 25
```

或者，使用您喜欢的邮件客户端（如 Thunderbird、Evolution 等）测试通过 IMAP 或 POP3 连接。

## 结论

您现在拥有一个基础的、能够发送和接收电子邮件的邮件服务器。为了使其成为功能齐全且安全的邮件系统，您需要进一步保护和优化，包括添加反垃圾邮件和 Web 邮件客户端等组件。
