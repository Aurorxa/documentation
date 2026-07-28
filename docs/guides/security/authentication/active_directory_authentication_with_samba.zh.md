---
title: 使用 Samba 进行 Active Directory 认证
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.4
---

## 前提条件

- 对 Active Directory 有一定的了解
- 对 LDAP 有一定的了解

## 简介

在大多数企业中，Microsoft 的 Active Directory (AD) 是 Windows 系统以及外部的、LDAP 连接服务的默认认证系统。它允许您配置用户和组、访问控制、权限、自动挂载等。

虽然将 Linux 连接到 AD 集群不能支持上述*所有*功能，但它可以处理用户、组和访问控制。通过 Linux 端的一些配置调整和 AD 端的一些高级选项，可以使用 AD 分发 SSH 密钥。

在 Rocky Linux 上使用 Active Directory 的默认方式是使用 SSSD，但 Samba 是一个功能更丰富的替代方案。例如，文件共享可以使用 Samba 但不能使用 SSSD。但是，本指南将涵盖使用 Samba 配置针对 Active Directory 的认证，不包括 Windows 端的任何额外配置。

## 使用 Samba 发现和加入 AD

!!! Note

    本指南中的域名 `ad.company.local` 将代表 Active Directory 域。要遵循本指南，请将其替换为您 AD 域的名称。

将 Linux 系统加入 AD 的第一步是发现您的 AD 集群，以确保两端的网络配置正确。

### 准备工作

- 确保以下端口在您的域控制器上对 Linux 主机开放：

  | 服务  | 端口           | 说明                                                       |
  |----------|-------------------|-------------------------------------------------------------|
  | DNS      | 53 (TCP+UDP)      |                                                             |
  | Kerberos | 88, 464 (TCP+UDP) | 由 `kadmin` 用于设置和更新密码           |
  | LDAP     | 389 (TCP+UDP)     |                                                             |
  | LDAP-GC  | 3268 (TCP)        | LDAP 全局目录 - 允许您从 AD 获取用户 ID |

- 确保您已将 AD 域控制器配置为 Rocky Linux 主机上的 DNS 服务器：

  **使用 NetworkManager：**

  ```sh
  # 其中您的主要 NetworkManager 连接是 'System eth0'，您的 AD
  # 服务器可通过 IP 地址 10.0.0.2 访问。
  [root@host ~]$ nmcli con mod 'System eth0' ipv4.dns 10.0.0.2
  ```
  
- 确保两端（AD 主机和 Linux 系统）的时间是同步的（参见 chronyd）

  **在 Rocky Linux 上检查时间：**

  ```sh
  [user@host ~]$ date
  Wed 22 Sep 17:11:35 BST 2021
  ```

- 在 Linux 端安装 AD 连接所需的软件包：

  ```sh
  [user@host ~]$ sudo dnf install samba samba-winbind samba-client
  ```

### 发现

现在您应该能够成功地从 Linux 主机发现您的 AD 服务器。

```sh
[user@host ~]$ realm discover ad.company.local
ad.company.local
  type: kerberos
  realm-name: AD.COMPANY.LOCAL
  domain-name: ad.company.local
  configured: no
  server-software: active-directory
  client-software: sssd
  required-package: oddjob
  required-package: oddjob-mkhomedir
  required-package: sssd
  required-package: adcli
  required-package: samba-common
```

存储在您 Active Directory DNS 服务中的相关 SRV 记录将允许发现。

### 加入

一旦您成功从 Linux 主机发现了您的 Active Directory 安装，您应该能够使用 `realmd` 加入域，它将通过 `adcli` 和其他一些类似工具协调 Samba 的配置。

```sh
[user@host ~]$ sudo realm join -v --membership-software=samba --client-software=winbind ad.company.local
```

您将被提示输入域的管理员密码，请输入。

如果此过程抱怨加密问题并显示 `KDC has no support for encryption type`，请尝试更新全局加密策略以允许较旧的加密算法：

```sh
[user@host ~]$ sudo update-crypto-policies --set DEFAULT:AD-SUPPORT
```

如果此过程成功，您现在应该能够获取 Active Directory 用户的 `passwd` 信息。

```sh
[user@host ~]$ sudo getent passwd administrator@ad.company.local
AD\administrator:*:1450400500:1450400513:Administrator:/home/administrator@ad.company.local:/bin/bash
```

!!! Note

    `getent` 从名称服务切换库（NSS）获取条目。这意味着，与 `passwd` 或 `dig` 等命令不同，它将查询不同的数据库，包括在 `getent hosts` 的情况下查询 `/etc/hosts`，或在 `getent passwd` 的情况下从 `samba` 查询。

`realm` 提供了一些可以使用的有趣选项：

| 选项                                                     | 说明                              |
|------------------------------------------------------------|------------------------------------------|
| --computer-ou='OU=LINUX,OU=SERVERS,dc=ad,dc=company.local' | 存储服务器账户的 OU |
| --os-name='rocky'                                          | 指定存储在 AD 中的操作系统名称     |
| --os-version='8'                                           | 指定存储在 AD 中的操作系统版本  |
| -U admin_username                                          | 指定管理员账户                 |

### 尝试认证

现在您的用户应该能够针对 Active Directory 在您的 Linux 主机上进行认证。

**在 Windows 10 上：**（提供其自带的 OpenSSH 副本）

```dos
C:\Users\John.Doe> ssh -l john.doe@ad.company.local linux.host
Password for john.doe@ad.company.local:

Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed Sep 15 17:37:03 2021 from 10.0.10.241
[john.doe@ad.company.local@host ~]$
```

如果成功，您已成功配置 Linux 使用 Active Directory 作为认证源。

### 消除用户名中的域名

在完全默认的设置中，您需要通过用户名中指定域来登录您的 AD 账户（例如 `john.doe@ad.company.local`）。如果这不是期望的行为，您希望在认证时可以省略默认域名，您可以配置 Samba 默认为特定域。

这是一个相对简单的过程，需要在您的 `smb.conf` 配置文件中进行配置调整。

```sh
[user@host ~]$ sudo vi /etc/samba/smb.conf
[global]
...
winbind use default domain = yes
```

通过添加 `winbind use default domain`，您指示 Samba 推断用户正在尝试以 `ad.company.local` 域中的用户身份进行认证。这允许您以 `john.doe` 而不是 `john.doe@ad.company.local` 的方式认证。

要使此配置更改生效，您必须使用 `systemctl` 重启 `smb` 和 `winbind` 服务。

```sh
[user@host ~]$ sudo systemctl restart smb winbind
```

同样，如果您不希望您的主目录后缀加上域名，可以将这些选项添加到您的配置文件 `/etc/samba/smb.conf` 中：

```bash
[global]
template homedir = /home/%U
```

不要忘记重启 `smb` 和 `winbind` 服务。
