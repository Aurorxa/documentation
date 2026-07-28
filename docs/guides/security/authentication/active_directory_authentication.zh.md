---
title: Active Directory 认证 
author: Hayden Young
contributors: Steven Spencer, Sambhav Saggi, Antoine Le Morvan, Krista Burdine, Ganna Zhyrnova, Neel Chauhan
tested_with: 9.4
---

## 前提条件

- 对 Active Directory 有一定的了解
- 对 LDAP 有一定的了解

## 简介

在大多数企业中，Microsoft 的 Active Directory (AD) 是 Windows 系统以及外部的、LDAP 连接服务的默认认证系统。它允许您配置用户和组、访问控制、权限、自动挂载等。

虽然将 Linux 连接到 AD 集群不能支持上述*所有*功能，但它可以处理用户、组和访问控制。通过 Linux 端的一些配置调整和 AD 端的一些高级选项，可以使用 AD 分发 SSH 密钥。

但是，本指南仅涵盖配置针对 Active Directory 的认证，不包括 Windows 端的任何额外配置。

## 使用 SSSD 发现和加入 AD

!!! Note

    本指南中的域名 `ad.company.local` 将代表 Active Directory 域。要遵循本指南，请将其替换为您 AD 域的实际域名。

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
  [user@host ~]$ sudo dnf install realmd oddjob oddjob-mkhomedir sssd adcli krb5-workstation
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

一旦您成功从 Linux 主机发现了您的 Active Directory 安装，您应该能够使用 `realmd` 加入域，它将通过 `adcli` 和其他一些类似工具协调 `sssd` 的配置。

```sh
[user@host ~]$ sudo realm join ad.company.local
```

如果此过程抱怨加密问题并显示 `KDC has no support for encryption type`，请尝试更新全局加密策略以允许较旧的加密算法：

```sh
[user@host ~]$ sudo update-crypto-policies --set DEFAULT:AD-SUPPORT
```

如果此过程成功，您现在应该能够获取 Active Directory 用户的 `passwd` 信息。

```sh
[user@host ~]$ sudo getent passwd administrator@ad.company.local
administrator@ad.company.local:*:1450400500:1450400513:Administrator:/home/administrator@ad.company.local:/bin/bash
```

!!! Note

    `getent` 从名称服务切换库（NSS）获取条目。这意味着，与 `passwd` 或 `dig` 等命令不同，它将查询不同的数据库，包括在 `getent hosts` 的情况下查询 `/etc/hosts`，或在 `getent passwd` 的情况下从 `sssd` 查询。

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

### 设置默认域

在完全默认的设置中，您需要通过用户名中指定域来登录您的 AD 账户（例如 `john.doe@ad.company.local`）。如果这不是期望的行为，您希望在认证时可以省略域名，您可以配置 SSSD 默认为特定域。

这是一个相对简单的过程，需要在您的 SSSD 配置文件中进行配置调整。

```sh
[user@host ~]$ sudo vi /etc/sssd/sssd.conf
[sssd]
...
default_domain_suffix = ad.company.local
```

通过添加 `default_domain_suffix`，您指示 SSSD（如果没有指定其他域）推断用户正在尝试以 `ad.company.local` 域中的用户身份进行认证。这允许您以 `john.doe` 而不是 `john.doe@ad.company.local` 的方式认证。

要使此配置更改生效，您必须使用 `systemctl` 重启 `sssd.service` 单元。

```sh
[user@host ~]$ sudo systemctl restart sssd
```

同样，如果您不希望您的主目录后缀加上域名，可以将这些选项添加到您的配置文件 `/etc/sssd/sssd.conf` 中：

```
[domain/ad.company.local]
use_fully_qualified_names = False
override_homedir = /home/%u
```

不要忘记重启 `sssd` 服务。

### 限制特定用户

有各种方法可以限制服务器访问到有限的用户列表，但这种方法，顾名思义，当然是最简单的：

将这些选项添加到您的配置文件 `/etc/sssd/sssd.conf` 中并重启服务：

```
access_provider = simple
simple_allow_groups = group1, group2
simple_allow_users = user1, user2
```

现在，只有 group1 和 group2 中的用户，或者 user1 和 user2 能够使用 sssd 连接到服务器！

## 使用 `adcli` 与 AD 交互

`adcli` 是一个用于在 Active Directory 域上执行操作的 CLI（命令行界面）工具。

- 如果尚未安装，请安装所需的软件包：

```sh
[user@host ~]$ sudo dnf install adcli
```

- 测试您是否曾经加入过 Active Directory 域：

```sh
[user@host ~]$ sudo adcli testjoin
Successfully validated join to domain ad.company.local
```

- 获取有关域的更高级信息：

```sh
[user@host ~]$ adcli info ad.company.local
[domain]
domain-name = ad.company.local
domain-short = AD
domain-forest = ad.company.local
domain-controller = dc1.ad.company.local
domain-controller-site = site1
domain-controller-flags = gc ldap ds kdc timeserv closest writable full-secret ads-web
domain-controller-usable = yes
domain-controllers = dc1.ad.company.local dc2.ad.company.local
[computer]
computer-site = site1
```

- Adcli 不仅仅是一个咨询工具，您可以使用 adcli 与您的域交互：管理用户或组、更改密码等。

示例：使用 `adcli` 获取有关计算机的信息：

!!! Note

    这次我们将通过 `-U` 选项提供管理员用户名

```sh
[user@host ~]$ adcli show-computer pctest -U admin_username
Password for admin_username@AD: 
sAMAccountName:
 pctest$
userPrincipalName:
 - not set -
msDS-KeyVersionNumber:
 9
msDS-supportedEncryptionTypes:
 24
dNSHostName:
 pctest.ad.company.local
servicePrincipalName:
 RestrictedKrbHost/pctest.ad.company.local
 RestrictedKrbHost/pctest
 host/pctest.ad.company.local
 host/pctest
operatingSystem:
 Rocky
operatingSystemVersion:
 8
operatingSystemServicePack:
 - not set -
pwdLastSet:
 133189248188488832
userAccountControl:
 69632
description:
 - not set -
```

示例：使用 `adcli` 更改用户密码：

```sh
[user@host ~]$ adcli passwd-user user_test -U admin_username
Password for admin_username@AD: 
Password for user_test: 
[user@host ~]$ 
```

## 故障排除

有时，网络服务会在 SSSD 之后启动，这会导致认证出现问题。

在您重启服务之前，没有 AD 用户能够连接。

在这种情况下，您将需要覆盖 systemd 的服务文件来处理此问题。

将此内容复制到 `/etc/systemd/system/sssd.service`：

```
[Unit]
Description=System Security Services Daemon
# SSSD must be running before we permit user sessions
Before=systemd-user-sessions.service nss-user-lookup.target
Wants=nss-user-lookup.target
After=network-online.target


[Service]
Environment=DEBUG_LOGGER=--logger=files
EnvironmentFile=-/etc/sysconfig/sssd
ExecStart=/usr/sbin/sssd -i ${DEBUG_LOGGER}
Type=notify
NotifyAccess=main
PIDFile=/var/run/sssd.pid

[Install]
WantedBy=multi-user.target
```

下次重启时，服务将在其依赖项之后启动，一切都会顺利进行。

## 离开 Active Directory

有时，有必要离开 AD。

您可以再次使用 `realm` 进行操作，然后移除不再需要的软件包：

```sh
[user@host ~]$ sudo realm leave ad.company.local
[user@host ~]$ sudo dnf remove realmd oddjob oddjob-mkhomedir sssd adcli krb5-workstation
```
