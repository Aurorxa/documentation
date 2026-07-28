---
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
title: 第 4.1 部分 数据库服务器 MariaDB
tags:
  - mariadb
  - mysql
  - database
  - rdbms

---

MySQL、MariaDB 和 PostgreSQL 是开源 RDBMS（Relational DataBase Management System，关系型数据库管理系统）。

## MariaDB 和 MySQL

在本章中，您将学习关于 RDBMS MariaDB 和 MySQL 的知识。

****

**目标**：您将学习如何：

:heavy_check_mark: 安装、配置并保护 MariaDB 服务器和 MySQL 服务器；  
:heavy_check_mark: 对数据库和用户执行一些管理操作。  

:checkered_flag: **RDBMS**, **database**, **MariaDB**, **MySQL**

**知识**：:star: :star: :star:  
**复杂性**：:star: :star: :star:  

**阅读时间**：30 分钟

****

### 概述

MySQL 由 Michael "Monty" Widenius（一位芬兰计算机科学家）开发，他于 1995 年创立了 MySQL AB。MySQL AB 于 2008 年被 SUN 收购，SUN 随后于 2009 年被 Oracle 收购。Oracle 仍然拥有 MySQL 软件，并以 GPL 和专有双许可证进行分发。

2009 年，Michael Widenius 离开 SUN，创立了 Monty Program AB，并在 GPL 许可下启动了他的 MySQL 社区分支的开发：MariaDB。MariaDB 基金会管理该项目，并确保其保持自由。

不久之后，大多数 Linux 发行版开始提供 MariaDB 包而非 MySQL 包，维基百科和 Google 等重要客户也采用了这个社区分支。

MySQL 和 MariaDB 是全球使用最广泛的 RDBMS 之一（专业和普通大众），特别是用于 Web 应用（**LAMP**：Linux + Apache + Mysql-MariaDB + Php）。

Mysql-MariaDB 的主要竞争对手是：

* PostgreSQL,
* OracleDB,
* Microsoft SQL Server。

数据库服务是多线程和多用户的，运行在大多数操作系统（Linux、Unix、BSD、Mac OSx、Windows）上，可从许多编程语言（PHP、Java、Python、C、C++、Perl 等）访问。

支持多种引擎，允许根据需求将不同的引擎分配给同一数据库中的不同表：

MyISAM
:   最简单的，但不支持事务或外键。它是一个索引顺序引擎。MyISAM 现已弃用。

InnoDB
:   管理表完整性（外键和事务），但占用更多磁盘空间。自 MySQL 5.6 版本以来，这已是默认引擎。它是一个事务引擎。

Memory
:   表存储在内存中。

Archive
:   插入时进行数据压缩，节省磁盘空间但减慢搜索查询（冷数据）。

这是根据需求采用引擎的问题：Archive 用于日志存储，Memory 用于临时数据，等等。

MariaDB/MySQL 使用 3306/TCP 端口进行网络通信。

本章将讨论此版本，因为 Rocky 提供的默认版本是数据库的 MariaDB 社区版本。仅特别处理 MySQL 和 MariaDB 之间的差异。

### 安装

使用 `dnf` 命令安装 `mariadb-server` 包：

```bash
sudo dnf install -y mariadb-server
```

默认情况下，Rocky 9 上安装的版本是 10.5。

激活服务在启动时运行并启动它：

```bash
sudo systemctl enable mariadb --now
```

您可以检查 `mariadb` 服务的状态：

```bash
sudo systemctl status mariadb
```

要安装更新的版本，您需要使用 `dnf` modules：

```bash
$ sudo dnf module list mariadb
Last metadata expiration check: 0:00:09 ago on Thu Jun 20 11:39:10 2024.
Rocky Linux 9 - AppStream
Name                          Stream                      Profiles                                        Summary
mariadb                       10.11                       client, galera, server [d]                      MariaDB Module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

如果您尚未安装 mariadb 服务器，激活所需的 module 版本即可：

```bash
$ sudo dnf module enable mariadb:10.11
Last metadata expiration check: 0:02:23 ago on Thu Jun 20 11:39:10 2024.
Dependencies resolved.
============================================================================================================================================= Package                          Architecture                    Version                             Repository                        Size
=============================================================================================================================================
Enabling module streams:
 mariadb                                                          10.11

Transaction Summary
=============================================================================================================================================
Is this ok [y/N]: y
Complete!
```

现在您可以安装包了。所需版本将自动安装：

```bash
sudo dnf install -y mariadb-server
```

#### 关于默认用户

请注意 mariadb 首次启动时提供的日志（`/var/log/messages`）：

```text
mariadb-prepare-db-dir[6560]: Initializing MariaDB database
mariadb-prepare-db-dir[6599]: Two all-privilege accounts were created.
mariadb-prepare-db-dir[6599]: One is root@localhost, it has no password, but you need to
mariadb-prepare-db-dir[6599]: be system 'root' user to connect. Use, for example, sudo mysql
mariadb-prepare-db-dir[6599]: The second is mysql@localhost, it has no password either, but
mariadb-prepare-db-dir[6599]: you need to be the system 'mysql' user to connect.
mariadb-prepare-db-dir[6599]: After connecting you can set the password, if you would need to be
mariadb-prepare-db-dir[6599]: able to connect as any of these users with a password and without sudo
```

### 配置

配置文件可以在 `/etc/my.cnf` 和 `/etc/my.cnf.d/` 中找到。

一些重要的默认选项已在 `/etc/my.cnf.d/mariadb-server.cnf` 中设置：

```text
[server]

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mariadb/mariadb.log
pid-file=/run/mariadb/mariadb.pid
...
```

如您所见，数据默认位于 `/var/lib/mysql`。此文件夹可能需要大量存储空间和反复的容量增长。因此，建议将此文件夹挂载到专用分区上。

### 安全

MariaDB 和 Mysql 包含一个脚本，帮助您保护服务器。例如，它移除了远程 root 登录和示例用户，以及不太安全的默认选项。

使用 `mariadb-secure-installation` 保护您的服务器：

```bash
sudo mariadb-secure-installation
```

该脚本将提示您为 root 用户提供密码。

!!! NOTE

    `mysql_secure_installation` 命令现在是 `mariadb-secure-installation` 命令的符号链接：

    ```bash
    $ ll /usr/bin/mysql_secure_installation
    lrwxrwxrwx. 1 root root 27 Oct 12  2023 /usr/bin/mysql_secure_installation -> mariadb-secure-installation
    ```

如果每次使用 mariadb 命令时都要提供密码是个问题，您可以设置一个 `~/.my.cnf` 文件，其中包含您的凭据，mariadb 将默认使用该凭据连接到您的服务器。

```bash
[client]
user="root"
password="#######"
```

确保权限足够严格，只允许当前用户访问：

```bash
chmod 600 ~/.my.cnf
```

!!! WARNING

    这不是最好的方式。有另一种比明文存储密码更安全的解决方案。自 MySQL 5.6.6 起，现在可以通过 `mysql_config_editor` 命令将凭据存储在加密的登录文件 `.mylogin.cnf` 中。

如果您的服务器运行防火墙（这是一件好事），您可能需要考虑开放它，但仅在您需要从外部访问您的服务时。

```bash
sudo firewall-cmd --zone=public --add-service=mysql
sudo firewall-cmd --reload
```

!!! NOTE

    最佳安全实践是不要向外部开放您的数据库服务器（如果应用服务器托管在同一台服务器上），或者仅限制对授权 IP 的访问。

### 管理

#### `mariadb` 命令

`mariadb` 命令是一个简单的 SQL shell，支持交互式和非交互式使用。

```bash
mysql -u user -p [base]
```

| 选项    | 信息                          |
| --------- | ------------------------------------ |
| `-u user` | 提供用于连接的用户名。 |
| `-p`      | 要求输入密码。                 |
| `base`    | 要连接到的数据库。          |

!!! NOTE

    `mysql` 命令现在是 `mariadb` 命令的符号链接：

    ```bash
    $ ll /usr/bin/mysql
    lrwxrwxrwx. 1 root root 7 Oct 12  2023 /usr/bin/mysql -> mariadb
    ```

示例：

```bash
$ sudo mariadb -u root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 15
Server version: 10.5.22-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.003 sec)
```

#### `mariadb-admin` 命令

`mariadb-admin` 命令是用于管理 MariaDB 服务器的客户端。
  
```bash
mariadb-admin -u user -p command
```

| 选项    | 信息                          |
| --------- | ------------------------------------ |
| `-u user` | 提供用于连接的用户名。 |
| `-p`      | 要求输入密码。                 |
| `command` | 要执行的命令。                |

`mariadb-admin` 提供多个命令，如 `version`、`variables`、`stop-slave` 或 `start-slaves`、`create databasename` 等。

示例：

```bash
mariadb-admin -u root -p version
```

!!! NOTE

    `mysqladmin` 命令现在是 `mariadb-admin` 命令的符号链接：

    ```bash
    $ ll /usr/bin/mysqladmin
    lrwxrwxrwx. 1 root root 13 Oct 12  2023 /usr/bin/mysqladmin -> mariadb-admin
    ```

### 关于日志

MariaDB 提供各种日志：

* **错误日志（Error log）**：包含服务启动和关闭时生成的消息以及重要事件（警告和错误）。
* **二进制日志（Binary log）**：此日志（二进制格式）记录所有修改数据库结构或数据的行为。如果您需要恢复数据库，您需要恢复备份并重放二进制日志以恢复崩溃前数据库的状态。
* **查询日志（Query log）**：所有客户端请求都记录在此处。
* **慢请求日志（Slow requests log）**：慢查询，即那些执行时间超过设定时间的查询，单独记录在此日志中。通过分析此文件，您可能能够采取措施减少执行时间（例如，通过设置索引或修改客户端应用程序）。

除二进制日志外，这些日志都是文本格式，因此可以直接使用！

要启用长请求的日志记录，编辑 `my.cnf` 配置文件，添加以下行：

```bash
slow_query_log      = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time     = 2
```

`long_query_time` 变量的最小值为 0，默认值为 `10` 秒。

重启服务以使更改生效。

一旦日志文件满了，您可以使用 `mariadb-dumpslow` 命令进行分析。

```bash
mariadb-dumpslow [options] [log_file ...]
```

| 选项         | 信息                          |
| -------------- | ------------------------------------ |
| `-t n`         | 仅显示前 n 个查询。   |
| `-s sort_type` | 按查询数量排序。          |
| `-r`           | 反转结果显示。             |

排序类型可以是：

| 选项      | 信息                                                            |
| ----------- | ---------------------------------------------------------------------- |
| `c`         | 根据请求数量。                                       |
| `t` 或 `at` | 根据执行时间或平均执行时间（a 表示平均）。 |
| `l` 或 `al` | 根据锁定时间或其平均值。                                 |
| `r` 或 `aR` | 根据返回行数或其平均值的函数。          |

### 关于备份

与任何 RDBMS 一样，备份数据库是在数据修改离线时进行的。您可以通过以下方式进行：

* 停止服务，称为离线备份；
* 在服务运行时通过临时锁定更新（暂停所有修改）进行。这是在线备份。
* 使用 LVM 文件系统的快照，实现冷文件系统的数据备份。

备份格式可以是 ASCII（文本）文件，以 SQL 命令形式表示数据库及其数据的状态，或对应于 MySQL 存储文件的二进制文件。

虽然您可以使用常见工具如 tar 或 cpio 备份二进制文件，但 ASCII 文件则需要类似 `mariadb-dump` 的工具。

`mariadb-dump` 命令可以执行数据库的转储（dump）。

在此过程中，数据访问被锁定。

```bash
mariadb-dump -u root -p DATABASE_NAME > backup.sql
```

!!! NOTE

    不要忘记，在恢复完整备份后，恢复二进制文件（binlogs）才能完成数据的重建。

生成的文件可用于恢复数据库数据。数据库必须仍然存在，或者您必须事先重新创建它！：

```bash
mariadb -u root -p DATABASE_NAME < backup.sql
```

### 图形化工具

存在图形化工具来简化数据库数据的管理与管理。以下是一些示例：

* [DBeaver](https://dbeaver.io/)

### 实践坊

在本实践坊中，您将安装、配置和保护您的 MariaDB 服务器。

#### 任务 1：安装

安装 MariaDB-server 包：

```bash
$ sudo dnf install mariadb-server
Last metadata expiration check: 0:10:05 ago on Thu Jun 20 11:26:03 2024.
Dependencies resolved.
============================================================================================================================================= Package                                       Architecture            Version                              Repository                  Size
=============================================================================================================================================
Installing:
 mariadb-server                                x86_64                  3:10.5.22-1.el9_2                    appstream                  9.6 M
Installing dependencies:
...
```

安装会向系统添加一个 `mysql` 用户，其主目录为 `/var/lib/mysql`：

```bash
$ cat /etc/passwd
...
mysql:x:27:27:MySQL Server:/var/lib/mysql:/sbin/nologin
...
```

启用并启动服务：

```bash
$ sudo systemctl enable mariadb --now
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
```

检查安装：

```bash
$ sudo systemctl status mariadb
● mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: disabled)
     Active: active (running) since Thu 2024-06-20 11:48:56 CEST; 1min 27s ago
       Docs: man:mariadbd(8)
             https://mariadb.com
    Process: 6538 ExecStartPre=/usr/libexec/mariadb-check-socket (code=exited, status=0/SUCCESS)
    Process: 6560 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir mariadb.service (code=exited, status=0/SUCCESS)
    Process: 6658 ExecStartPost=/usr/libexec/mariadb-check-upgrade (code=exited, status=0/SUCCESS)
   Main PID: 6643 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 9 (limit: 11110)
     Memory: 79.5M
        CPU: 1.606s
     CGroup: /system.slice/mariadb.service
             └─6643 /usr/libexec/mariadbd --basedir=/usr

Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: The second is mysql@localhost, it has no password either, but
Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: you need to be the system 'mysql' user to connect.
Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: After connecting you can set the password, if you would need to be
Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: able to connect as any of these users with a password and without sudo
Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: See the MariaDB Knowledgebase at https://mariadb.com/kb
Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: Please report any problems at https://mariadb.org/jira
Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: The latest information about MariaDB is available at https://mariadb.org>Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: Consider joining MariaDB's strong and vibrant community:
Jun 20 11:48:56 localhost.localdomain mariadb-prepare-db-dir[6599]: https://mariadb.org/get-involved/
Jun 20 11:48:56 localhost.localdomain systemd[1]: Started MariaDB 10.5 database server.
```

尝试连接到服务器：

```bash
$ sudo mariadb
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.5.22-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.001 sec)

MariaDB [(none)]> exit
Bye
```

```bash
$ sudo mariadb-admin version
mysqladmin  Ver 9.1 Distrib 10.5.22-MariaDB, for Linux on x86_64
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Server version          10.5.22-MariaDB
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
Uptime:                 7 min 24 sec

Threads: 1  Questions: 9  Slow queries: 0  Opens: 17  Open tables: 10  Queries per second avg: 0.020
```

如您所见，`root` 用户不需要提供密码。您将在下一个任务中纠正这一点。

#### 任务 2：保护您的服务器

启动 `mariadb-secure-installation` 并按照说明操作：

```bash
$ sudo mariadb-secure-installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] y
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

再次尝试连接，分别使用和不使用密码连接到您的服务器：

```bash
$ mariadb -u root
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

$ mariadb -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.5.22-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

配置您的防火墙：

```bash
sudo firewall-cmd --zone=public --add-service=mysql --permanent
sudo firewall-cmd --reload
```

#### 任务 3：测试安装

验证您的安装：

```bash
$ mysqladmin -u root -p version
Enter password:
mysqladmin  Ver 9.1 Distrib 10.5.22-MariaDB, for Linux on x86_64
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Server version          10.5.22-MariaDB
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
Uptime:                 29 min 18 sec

Threads: 1  Questions: 35  Slow queries: 0  Opens: 20  Open tables: 13  Queries per second avg: 0.019
```

`version` 为您提供有关服务器的信息。

#### 任务 4：创建新的数据库和用户

创建一个新数据库：

```sql
MariaDB [(none)]> create database NEW_DATABASE_NAME;
```

创建一个新用户并授予其对数据库所有表的所有权限：

```sql
MariaDB [(none)]> grant all privileges on NEW_DATABASE_NAME.* TO 'NEW_USER_NAME'@'localhost' identified by 'PASSWORD';
```

如果您想授予从任意位置的访问权限，请将 `localhost` 替换为 `%`，或者在可能的情况下替换为 IP 地址。

您可以限制授予的权限。有多种不同类型的权限可以提供給用户：

* **SELECT**：读取数据
* **USAGE**：授权连接到服务器（在创建新用户时默认授予）
* **INSERT**：向表中添加新元组。
* **UPDATE**：修改现有元组
* **DELETE**：删除元组
* **CREATE**：创建新表或数据库
* **DROP**：删除现有表或数据库
* **ALL PRIVILEGES**：所有权限
* **GRANT OPTION**：给予或移除其他用户的权限

不要忘记重新加载并应用新的权限：

```sql
MariaDB [(none)]> flush privileges;
```

检查：

```bash
$ mariadb -u NEW_USER_NAME -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.5.22-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| NEW_DATABASE_NAME  |
| information_schema |
+--------------------+
2 rows in set (0.001 sec)

```

向您的数据库添加示例数据：

```bash
$ mariadb -u NEW_USER_NAME -p NEW_DATABASE_NAME
MariaDB [NEW_DATABASE_NAME]> CREATE TABLE users(
    id INT NOT NULL AUTO_INCREMENT,
    first_name VARCHAR(30) NOT NULL,
    last_name VARCHAR(30) NOT NULL,
    age INT DEFAULT NULL,
    PRIMARY KEY (id));
Query OK, 0 rows affected (0.017 sec)

MariaDB [NEW_DATABASE_NAME]> INSERT INTO users (first_name, last_name, age) VALUES ("Antoine", "Le Morvan", 44);
Query OK, 1 row affected (0.004 sec)
```

#### 任务 5：创建远程用户

在此任务中，您将创建一个新用户，授予远程访问权限，并使用该用户测试连接。

```bash
MariaDB [(none)]> grant all privileges on NEW_DATABASE_NAME.* TO 'NEW_USER_NAME'@'%' identified by 'PASSWORD';
Query OK, 0 rows affected (0.005 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.004 sec)
```

使用此用户和 `-h` 选项远程连接到您的服务器：

```bash
$ mariadb -h YOUR_SERVER_IP -u NEW_USER_NAME -p NEW_DATABASE_NAME
Enter password:
...

MariaDB [NEW_DATABASE_NAME]>
```

#### 任务 6：执行升级

启用所需的 module：

```bash
$ sudo dnf module enable mariadb:10.11
[sudo] password for antoine:
Last metadata expiration check: 2:00:16 ago on Thu Jun 20 11:50:27 2024.
Dependencies resolved.
============================================================================================================================================= Package                          Architecture                    Version                             Repository                        Size
=============================================================================================================================================Enabling module streams:
 mariadb                                                          10.11

Transaction Summary
=============================================================================================================================================
Is this ok [y/N]: y
Complete!
```

升级包：

```bash
$ sudo dnf update mariadb
Last metadata expiration check: 2:00:28 ago on Thu Jun 20 11:50:27 2024.
Dependencies resolved.
============================================================================================================================================= Package                            Architecture        Version                                                 Repository              Size
=============================================================================================================================================
Upgrading:
 mariadb                            x86_64              3:10.11.6-1.module+el9.4.0+20012+a68bdff7               appstream              1.7 M
 mariadb-backup                     x86_64              3:10.11.6-1.module+el9.4.0+20012+a68bdff7               appstream              6.7 M
 mariadb-common                     x86_64              3:10.11.6-1.module+el9.4.0+20012+a68bdff7               appstream               28 k
 mariadb-errmsg                     x86_64              3:10.11.6-1.module+el9.4.0+20012+a68bdff7               appstream              254 k
 mariadb-gssapi-server              x86_64              3:10.11.6-1.module+el9.4.0+20012+a68bdff7               appstream               15 k
 mariadb-server                     x86_64              3:10.11.6-1.module+el9.4.0+20012+a68bdff7               appstream               10 M
 mariadb-server-utils               x86_64              3:10.11.6-1.module+el9.4.0+20012+a68bdff7               appstream              261 k

Transaction Summary
=============================================================================================================================================
Upgrade  7 Packages

Total download size: 19 M
Is this ok [y/N]: y
Downloading Packages:
(1/7): mariadb-gssapi-server-10.11.6-1.module+el9.4.0+20012+a68bdff7.x86_64.rpm                               99 kB/s |  15 kB     00:00
(2/7): mariadb-server-utils-10.11.6-1.module+el9.4.0+20012+a68bdff7.x86_64.rpm                               1.1 MB/s | 261 kB     00:00
(3/7): mariadb-errmsg-10.11.6-1.module+el9.4.0+20012+a68bdff7.x86_64.rpm                                     2.5 MB/s | 254 kB     00:00
(4/7): mariadb-common-10.11.6-1.module+el9.4.0+20012+a68bdff7.x86_64.rpm                                     797 kB/s |  28 kB     00:00
(5/7): mariadb-10.11.6-1.module+el9.4.0+20012+a68bdff7.x86_64.rpm                                            5.7 MB/s | 1.7 MB     00:00
(6/7): mariadb-server-10.11.6-1.module+el9.4.0+20012+a68bdff7.x86_64.rpm                                     9.5 MB/s |  10 MB     00:01
(7/7): mariadb-backup-10.11.6-1.module+el9.4.0+20012+a68bdff7.x86_64.rpm                                     7.7 MB/s | 6.7 MB     00:00
---------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                         13 MB/s |  19 MB     00:01
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction

...

Complete!
```

您的数据库现在需要升级（检查您的 `/var/log/messages`，因为服务会抱怨）：

```text
mariadb-check-upgrade[8832]: The datadir located at /var/lib/mysql needs to be upgraded using 'mariadb-upgrade' tool. This can be done using the following steps:
mariadb-check-upgrade[8832]:  1. Back-up your data before with 'mariadb-upgrade'
mariadb-check-upgrade[8832]:  2. Start the database daemon using 'systemctl start mariadb.service'
mariadb-check-upgrade[8832]:  3. Run 'mariadb-upgrade' with a database user that has sufficient privileges
mariadb-check-upgrade[8832]: Read more about 'mariadb-upgrade' usage at:
mariadb-check-upgrade[8832]: https://mariadb.com/kb/en/mysql_upgrade/
```

不要忘记执行 MariaDB 提供的升级脚本：

```bash
sudo mariadb-upgrade
Major version upgrade detected from 10.5.22-MariaDB to 10.11.6-MariaDB. Check required!
Phase 1/8: Checking and upgrading mysql database
Processing databases
mysql
mysql.column_stats                                 OK
mysql.columns_priv                                 OK
mysql.db                                           OK
...
Phase 2/8: Installing used storage engines... Skipped
Phase 3/8: Running 'mysql_fix_privilege_tables'
Phase 4/8: Fixing views
mysql.user                                         OK
...
Phase 5/8: Fixing table and database names
Phase 6/8: Checking and upgrading tables
Processing databases
NEW_DATABASE_NAME
information_schema
performance_schema
sys
sys.sys_config                                     OK
Phase 7/8: uninstalling plugins
Phase 8/8: Running 'FLUSH PRIVILEGES'
OK
```

#### 任务 6：执行转储（dump）

`mariadb-dump` 命令可以执行数据库的转储。

```bash
mariadb-dump -u root -p NEW_DATABASE_NAME > backup.sql
```

验证：

```bash
cat backup.sql
-- MariaDB dump 10.19  Distrib 10.11.6-MariaDB, for Linux (x86_64)
--
-- Host: localhost    Database: NEW_DATABASE_NAME
-- ------------------------------------------------------
-- Server version       10.11.6-MariaDB

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
...

--
-- Table structure for table `users`
--

DROP TABLE IF EXISTS `users`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(30) NOT NULL,
  `last_name` varchar(30) NOT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=latin1 COLLATE=latin1_swedish_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `users`
--

LOCK TABLES `users` WRITE;
/*!40000 ALTER TABLE `users` DISABLE KEYS */;
INSERT INTO `users` VALUES
(1,'Antoine','Le Morvan',44);
/*!40000 ALTER TABLE `users` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

...
-- Dump completed on 2024-06-20 14:32:41
```

### 知识自测

:heavy_check_mark: 默认安装的数据库版本是哪个？

* [ ] MySQL 5.5
* [ ] MariaDB 10.5
* [ ] MariaDB 11.11
* [ ] Mysql 8

:heavy_check_mark: 使用哪个命令来应用权限更改？

* [ ] flush rights
* [ ] flush privileges
* [ ] mariadb reload
* [ ] apply

### 总结

在本章中，您安装并保护了 MariaDB 数据库服务器，并创建了数据库和专用用户。

这些技能是管理数据库的前置条件。

在下一节中，您将看到如何安装 MySQL 数据库而不是 MariaDB 分支。
