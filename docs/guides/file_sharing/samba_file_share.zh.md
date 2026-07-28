---
title: Samba Windows 文件共享
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - samba
  - 文件共享
  - windows
---

# Samba Windows 文件共享服务器

## 引言

Samba 是一个开源实现，提供与 Microsoft Windows 客户端无缝的文件和打印服务。它允许 Rocky Linux 服务器在 Windows 网络上充当文件共享服务器。Samba 使用 SMB/CIFS 协议，能够与 Windows 系统兼容，实现跨平台的互操作性。

Samba 可以提供以下功能：

* 在 Linux 和 Windows 系统之间共享文件和目录。
* 共享打印服务。
* 可以作为 Active Directory 域控制器，也可以作为成员服务器加入现有的 AD (Active Directory) 域。
* 提供名称解析和浏览服务。

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验
* 一个要共享的目录

## 安装 Samba

```bash
dnf install samba
```

## 配置 Samba 共享

创建要共享的目录：

```bash
mkdir -p /srv/samba/shared
```

更改目录权限：

```bash
chmod -R 755 /srv/samba/shared
chown -R nobody:nobody /srv/samba/shared
```

如果共享需要对用户进行身份验证，请创建 Samba 用户：

```bash
smbpasswd -a username
```

### `smb.conf` 配置

备份默认配置文件：

```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.orig
```

编辑 `/etc/samba/smb.conf` 文件，添加共享：

```text
[global]
    workgroup = WORKGROUP
    server string = Samba Server %v
    netbios name = rocky-linux
    security = user
    map to guest = bad user
    dns proxy = no

[shared]
    path = /srv/samba/shared
    browsable = yes
    writable = yes
    guest ok = yes
    read only = no
```

如果您需要特定用户的私有(Private)共享：

```text
[private]
    path = /srv/samba/private
    browsable = yes
    writable = yes
    guest ok = no
    valid users = username
```

保存后，测试配置文件的语法是否正确：

```bash
testparm
```

### 配置 SELinux

SELinux 会将 Samba 的共享操作限制在特定的布尔值中。您需要配置 SELinux 才能允许 Samba 共享。使用以下命令：

```bash
setsebool -P samba_enable_home_dirs on
setsebool -P samba_export_all_rw on
```

恢复 SELinux 上下文：

```bash
restorecon -R /srv/samba
```

### 配置防火墙

允许 Samba 通过防火墙：

```bash
firewall-cmd --add-service=samba --permanent
firewall-cmd --reload
```

启动并启用 Samba 服务：

```bash
systemctl enable --now smb nmb
```

## 从 Windows 访问

在 Windows 机器上，打开文件资源管理器(File Explorer)并输入以下内容：

```text
\\192.168.0.10
```

或者输入主机名：

```text
\\rocky-linux
```

之后，您应该能看到共享列表。

## 从 Linux 访问

在 Rocky Linux 客户端上，安装 `cifs-utils` 软件包：

```bash
dnf install cifs-utils
```

创建一个本地挂载点并挂载共享：

```bash
mkdir -p /mnt/samba
mount -t cifs //192.168.0.10/shared /mnt/samba -o username=username
```

## 结论

Samba 提供了一种简单而可靠的方式在 Rocky Linux 和 Windows（或其他支持 SMB 协议的系统）之间共享文件。其丰富的功能集和配置灵活性使其能够满足企业环境中的多种使用场景。
