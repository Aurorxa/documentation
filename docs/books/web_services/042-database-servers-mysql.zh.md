---
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
title: 第 4.2 部分 数据库服务器 MySQL
---

## MySQL

在本章中，您将学习如何安装 MySQL 服务器。

!!! NOTE

    仅包含 MariaDB 和 MySQL 版本之间的显著差异。

---

**目标**：您将学习如何：

:heavy_check_mark: 安装、配置并保护 MariaDB 服务器和 MySQL 服务器；

:checkered_flag: **RDBMS**, **database**, **MariaDB**, **MySQL**

**知识**：:star: :star: :star:  
**复杂性**：:star: :star: :star:  

**阅读时间**：10 分钟

---

### MySQL 安装

默认情况下，安装的 MySQL 版本是 8.0。

这次，您必须安装 `mysql-server` 包：

```bash
sudo dnf install mysql-server
```

并启动 `mysqld` 服务：

```bash
sudo systemctl enable mysqld.service --now
```

您现在可以按照前一章节进行操作，将以下命令替换为：

* `mariadb` => `mysql`
* `mariadb-admin` => `mysql_admin`
* `mariadb-dump` => `mysql_dump`
* `mariadb-secure-installation` => `mysql_secure_installation`

您需要使用不同的仓库来安装最新版本的 MySQL 服务器。

访问此页面：https://dev.mysql.com/downloads/repo/yum/ 并复制仓库 URL。

例如：

```bash
sudo dnf install -y https://dev.mysql.com/get/mysql84-community-release-el9-1.noarch.rpm
```

完成后，您可以执行 `dnf update`：

```bash
$ dnf update
Error: This command has to be run with superuser privileges (under the root user on most systems).
[antoine@localhost ~]$ sudo dnf update
MySQL 8.4 LTS Community Server                                                                               377 kB/s | 226 kB     00:00
MySQL Connectors Community                                                                                   110 kB/s |  53 kB     00:00
MySQL Tools 8.4 LTS Community                                                                                170 kB/s |  97 kB     00:00
Dependencies resolved.
============================================================================================================================================= Package                                   Architecture      Version                                Repository   
                        Size
=============================================================================================================================================Installing:
 mysql-community-client                    x86_64            8.4.0-1.el9                            mysql-8.4-lts-community            3.1 M
     replacing  mysql.x86_64 8.0.36-1.el9_3
 mysql-community-server                    x86_64            8.4.0-1.el9                            mysql-8.4-lts-community             50 M
     replacing  mariadb-connector-c-config.noarch 3.2.6-1.el9_0
     replacing  mysql-server.x86_64 8.0.36-1.el9_3
Installing dependencies:
  ...

Transaction Summary
=============================================================================================================================================Install  7 Packages

Total download size: 59 M
Is this ok [y/N]: y
Downloading Packages:
(1/7): mysql-community-client-plugins-8.4.0-1.el9.x86_64.rpm                                                 3.4 MB/s | 1.4 MB     00:00
(2/7): mysql-community-common-8.4.0-1.el9.x86_64.rpm                                                         1.3 MB/s | 576 kB     00:00
(3/7): mysql-community-icu-data-files-8.4.0-1.el9.x86_64.rpm                                                  30 MB/s | 2.3 MB     00:00
(4/7): mysql-community-client-8.4.0-1.el9.x86_64.rpm                                                         5.8 MB/s | 3.1 MB     00:00
(5/7): mysql-community-libs-8.4.0-1.el9.x86_64.rpm                                                           6.8 MB/s | 1.5 MB     00:00
(6/7): net-tools-2.0-0.62.20160912git.el9.x86_64.rpm                                                         1.1 MB/s | 292 kB     00:00
(7/7): mysql-community-server-8.4.0-1.el9.x86_64.rpm                                                          48 MB/s |  50 MB     00:01
---------------------------------------------------------------------------------------------------------------------------------------------Total                                                                                                         30 
MB/s |  59 MB     00:01
MySQL 8.4 LTS Community Server                                                                               3.0 MB/s | 3.1 kB     00:00
Importing GPG key 0xA8D3785C:
 Userid     : "MySQL Release Engineering <mysql-build@oss.oracle.com>"
 Fingerprint: BCA4 3417 C3B4 85DD 128E C6D4 B7B3 B788 A8D3 785C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-mysql-2023
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :
  ...
Installed:
  mysql-community-server-8.4.0-1.el9.x86_64
  ...

Complete!
```

不要忘记重新启用并重启您的服务器：

```bash
sudo systemctl enable mysqld.service --now
```

### MySQL 知识自测

:heavy_check_mark: 默认安装的 MySQL 数据库版本是哪个？

* [ ] MySQL 5.5
* [ ] MariaDB 10.5
* [ ] MariaDB 11.11
* [ ] Mysql 8
