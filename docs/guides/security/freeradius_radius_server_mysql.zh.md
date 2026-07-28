---
title: 使用 MariaDB 的 FreeRADIUS RADIUS 服务器
author: Neel Chauhan
contributors:
tested_with: 10.1
tags:
  - security
---

## 简介

RADIUS 是一个 AAA（认证、授权和计费，authentication, authorization, and accounting）协议，用于管理网络访问。[FreeRADIUS](https://www.freeradius.org/) 是 Linux 和其他类 Unix 系统上事实上的 RADIUS 服务器。

您可以让 FreeRADIUS 与 MariaDB 配合使用，例如用于 802.1X、Wi-Fi 或 VPN 认证。

## 前提条件与假设

此过程的最低要求如下：

* 能够以 root 用户身份运行命令或使用 `sudo` 提升权限
* 一个 MariaDB 服务器
* 一个 RADIUS 客户端，例如路由器、交换机或 Wi-Fi 接入点

## 安装 FreeRADIUS

您首先需要安装 EPEL 和 CRB：

```bash
dnf install epel-release
crb enable
```

然后您可以从 `dnf` 仓库安装 FreeRADIUS：

```bash
dnf install -y freeradius freeradius-mysql
```

## 安装 MariaDB

您需要安装 MariaDB：

```bash
dnf install mariadb-server
```

## 配置 FreeRADIUS

安装软件包后，您需要首先为 FreeRADIUS 生成 TLS 加密证书：

```bash
cd /etc/raddb/certs
./bootstrap
```

随后，您需要启用 `sql`。编辑 `/etc/raddb/sites-enabled/default` 文件，将 `-sql` 替换为 `sql`：

```bash
authorize {
   ...
   sql
   ...
}
...
accounting {
   ...
   sql
   ...
}
...
session {
   ...
   sql
   ...
}
...
post-auth {
    ...
    sql
    ...
    Post-Auth-Type REJECT {
        sql
    }
    ....

}
```

在 `/etc/raddb/sites-enabled/inner-tunnel` 中做同样的操作：

```bash
authorize {
   ...
   sql
   ...
}
...
session {
   ...
   sql
   ...
}
...
post-auth {
    ...
    sql
    ...
    Post-Auth-Type REJECT {
        sql
    }
    ....

}
```

在 `/etc/raddb/mods-available/sql` 文件中，将 `dialect` 更改为 `mysql`：

```bash
        dialect = "mysql"
```

然后更改 `driver`：

```bash
        driver = "rlm_sql_${dialect}"
```

在 `mysql {` 部分中，删除 `tls {` 子部分。

然后，设置数据库名称、用户名和密码：

```bash
        server = "127.0.0.1"
        port = 3306
        login = "radius"
        password = "password"
        ...
        radius_db = "radius"
```

将上述字段替换为您的相应服务器、用户名。

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

## 插入 MariaDB 架构

首先，启用 MariaDB 并运行设置。

```bash
systemctl enable --now mysql
mysql_secure_installation
```

接下来登录 MariaDB：

```bash
mysql -u root -p
```

现在创建 RADIUS 用户和数据库：

```bash
create database radius;
create user 'radius'@'localhost' identified by 'password';
grant all privileges on radius.* to 'radius'@'localhost';
```

将用户名、密码和数据库名称替换为所需的值。

然后插入 MariaDB 架构：

```bash
mysql -u root -p radius < /etc/raddb/mods-config/sql/dhcp/mysql/schema.sql
```

将数据库名称替换为您选择的名称。

## 创建用户

首先登录 MariaDB：

```bash
mysql -u root -p radius
```

然后您可以添加用户：

```bash
insert into radcheck (username,attribute,op,value) values("neha", "Cleartext-Password", ":=", "iloveicecream");
```

将 `neha` 和 `iloveicecream` 分别替换为所需的用户名和密码。

您也可以使用第三方软件添加用户。例如，WHMCS 和多种 ISP 计费系统允许这样做。

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
