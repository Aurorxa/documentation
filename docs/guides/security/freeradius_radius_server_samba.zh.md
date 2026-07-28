---
title: 使用 Samba Active Directory 的 FreeRADIUS RADIUS 服务器
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 10.1
tags:
  - security
---


## 简介

RADIUS 是一个 AAA（认证、授权和计费，authentication, authorization, and accounting）协议，用于管理网络访问。[FreeRADIUS](https://www.freeradius.org/) 是 Linux 和其他类 Unix 系统上事实上的 RADIUS 服务器。

您可以让 FreeRADIUS 与 Microsoft 的 Active Directory 配合使用，例如用于 802.1X、Wi-Fi 或 VPN 认证。

## 前提条件与假设

此过程的最低要求如下：

* 能够以 root 用户身份运行命令或使用 `sudo` 提升权限
* 一个 Active Directory 成员服务器，无论是使用 Windows Server 还是 Samba 域
* 一个 RADIUS 客户端，例如路由器、交换机或 Wi-Fi 接入点

## 配置 Samba

您需要[使用 Samba 配置 Active Directory](../security/authentication/active_directory_authentication_with_samba.md
)。请注意 sssd 将不起作用。

## 安装 FreeRADIUS

您可以从 `dnf` 仓库安装 FreeRADIUS：

```bash
dnf install -y freeradius
```

## 配置 FreeRADIUS

安装软件包后，您需要首先为 FreeRADIUS 生成 TLS 加密证书：

```bash
cd /etc/raddb/certs
./bootstrap
```

随后，您需要启用 `ntlm_auth`。编辑 `/etc/raddb/sites-enabled/default` 文件并在 `authenticate` 块中插入以下内容：

```bash
authenticate {
...
    ntlm_auth
...
}
```

在 `/etc/raddb/sites-enabled/inner_tunnel` 中插入相同内容：

```bash
authenticate {
...
    ntlm_auth
...
}
```

将 `/etc/raddb/mods-enabled/ntlm_auth` 中的 `program` 行更改为此：

```bash
    program = "/usr/bin/ntlm_auth --request-nt-key --domain=MYDOMAIN --username=%{mschap:User-Name} --password=%{User-Password}"
```

将 `MYDOMAIN` 替换为您的 Active Directory 域名。

您需要在 `/etc/raddb/mods-config/files/authorize` 中将 `ntlm_auth` 设置为默认认证类型。添加以下行：

```bash
DEFAULT   Auth-Type = ntlm_auth
```

您还需要定义客户端。这是为了防止未经授权访问我们的 RADIUS 服务器。编辑 `clients.conf` 文件：

```bash
vi clients.conf
```

插入以下内容：

```bash
client 172.20.0.254 {
        secret = secret123
}
```

将 `172.20.0.254` 和 `secret123` 替换为客户端将使用的 IP 地址和密钥值。为其他客户端重复此操作。

## 启用 MS-CHAP

MS-CHAP 允许通过 RADIUS 将哈希密码发送到 Active Directory。强烈推荐使用。

从 `/etc/raddb/mods-config/files/authorize` 中移除以下行：

```bash
DEFAULT   Auth-Type = ntlm_auth
```

然后在 `/etc/raddb/mods-enabled/mschap` 中插入以下行：

```bash
mschap {
...
        ntlm_auth = "/usr/bin/ntlm_auth --request-nt-key --allow-mschapv2 --username=%{mschap:User-Name:-None} --domain=%{%{mschap:NT-Domain}:-MYDOMAIN} --challenge=%{mschap:Challenge:-00} --nt-response=%{mschap:NT-Response:-00}"
...
}
```

将 `MYDOMAIN` 替换为您的 Active Directory 域名。

最后，将 `radiusd` 用户添加到 `wbpriv` 组：

```bash
usermod -a -G wbpriv radiusd
```

此步骤很重要，因为它允许 MS-CHAP 认证。

## 启用 FreeRADIUS

初始配置后，您可以启动 `radiusd`：

```bash
systemctl enable --now radiusd
```

## 在交换机上配置 RADIUS

设置 FreeRADIUS 服务器后，您将配置 RADIUS 客户端。

作为示例，作者的 MikroTik 交换机可以这样配置：

```bash
/radius
add address=172.20.0.12 secret=secret123 service=dot1x
/interface dot1x server
add interface=combo3
```

将 `172.20.0.12` 替换为 FreeRADIUS 服务器的 IP 地址，将 `secret123` 替换为您之前设置的密钥。
