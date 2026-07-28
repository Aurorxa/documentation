---
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
title: 第 4.3 部分 MariaDB 数据库复制
---

## MariaDB 辅助服务器

本章将教您如何配置 MariaDB 的 Primary/Secondary（主/从）系统服务器。

****

**目标**：您将学习如何：

:heavy_check_mark: 在您的服务器中激活 binlog；  
:heavy_check_mark: 设置辅助服务器以从主服务器复制数据。  

:checkered_flag: **MariaDB**, **Replication**, **Primary**, **Secondary**

**知识**：:star: :star:  
**复杂性**：:star: :star: :star:  

**阅读时间**：10 分钟

****

### MariaDB 辅助服务器概述

一旦您开始更密集地使用数据库，您必须在多个服务器上复制您的数据。

这可以通过几种方式实现：

* 将写请求分发到主服务器，将读请求分发到辅助服务器。
* 在辅助服务器上执行数据库备份，这避免了在备份期间阻塞主服务器上的写入。

如果您的使用变得更加苛刻，您可以考虑切换到 Primary/Primary（主/主）系统：复制以交叉方式进行，但要注意阻塞主键唯一性的风险。否则，您将需要切换到更高级的集群系统。

### MariaDB 辅助服务器配置

#### 如何激活 binlog

在主服务器和辅助服务器上执行此操作：

在 `[mariadb]` 键下，将以下选项添加到您的 `/etc/my.cnf.d/mariadb-server.cnf` 文件中：

```file
[mariadb]
log-bin
server_id=1
log-basename=server1
binlog-format=mixed
```

对于主服务器，以及对于辅助服务器：

```file
[mariadb]
log-bin
server_id=2
log-basename=server2
binlog-format=mixed
```

`server_id` 选项在集群中的每个服务器上必须是唯一的，而 `log-basename` 选项允许您为 binlog 文件指定前缀。如果您不这样做，您就不能重命名您的服务器。

现在您可以在两台服务器上重启 MariaDB 服务：

```bash
sudo systemctl restart mariadb
```

您可以检查 binlog 文件是否已正确创建：

```bash
$ ll /var/lib/mysql/
total 123332
...
-rw-rw----. 1 mysql mysql         0 Jun 21 11:07 multi-master.info
drwx------. 2 mysql mysql      4096 Jun 21 11:07 mysql
srwxrwxrwx. 1 mysql mysql         0 Jun 21 11:16 mysql.sock
-rw-rw----. 1 mysql mysql       330 Jun 21 11:16 server1-bin.000001
-rw-rw----. 1 mysql mysql        21 Jun 21 11:16 server1-bin.index
...
```

#### 如何配置复制

首先，在主服务器上，您需要创建授权复制数据的用户（注意限制授权的 IP）：

```bash
$ sudo mariadb

MariaDB [(none)]> CREATE USER 'replication'@'%' IDENTIFIED BY 'PASSWORD';
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
Query OK, 0 rows affected (0.002 sec)
```

或者为了更好的安全性（将 '192.168.1.101' 替换为您自己的辅助 IP）：

```bash
$ sudo mariadb

MariaDB [(none)]> CREATE USER 'replication'@'192.168.1.101' IDENTIFIED BY 'PASSWORD';
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'192.168.1.101';
Query OK, 0 rows affected (0.002 sec)
```

如果主服务器已包含数据，您必须锁定新事务。在此期间，数据的导出或导入会从辅助服务器进行，并告诉辅助服务器何时开始复制。如果您的服务器尚未包含任何数据，此过程将大大简化。

在查看二进制日志位置时，防止任何数据更改：

```bash
$ sudo mariadb

MariaDB [(none)]> FLUSH TABLES WITH READ LOCK;
Query OK, 0 rows affected (0.021 sec)

MariaDB [(none)]> SHOW MASTER STATUS;
+--------------------+----------+--------------+------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------------+----------+--------------+------------------+
| server1-bin.000001 |     1009 |              |                  |
+--------------------+----------+--------------+------------------+
1 row in set (0.000 sec)

```

不要退出您的会话以保持锁定。

记录 File 和 Position 的详细信息。

如果您的服务器包含数据，现在是创建备份并将其导入辅助服务器的时机。在备份期间保持锁定，备份完成后立即释放。这减少了停机时间（在辅助服务器上复制和导入数据所需的时间）。

您现在可以移除锁定了：

```bash
$ sudo mariadb

MariaDB [(none)]> UNLOCK TABLES;
Query OK, 0 rows affected (0.000 sec)
```

在辅助服务器上，您现在可以通过以下方式设置要复制的主服务器：

```bash
MariaDB [(none)]> CHANGE MASTER TO
  MASTER_HOST='192.168.1.100',
  MASTER_USER='replication',
  MASTER_PASSWORD='PASSWORD',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='server1-bin.000001',
  MASTER_LOG_POS=1009,
  MASTER_CONNECT_RETRY=10;
Query OK, 0 rows affected, 1 warning (0.021 sec)

MariaDB [(none)]> START SLAVE;
Query OK, 0 rows affected (0.001 sec)
```

将主服务器 IP 替换为您的 IP，将 `MASTER_LOG_FILE` 和 `MASTER_LOG_POS` 值替换为您之前记录的值。

检查复制是否正常：

```bash
MariaDB [(none)]> SHOW SLAVE STATUS \G
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: 192.168.1.100
                   Master_User: replication
               Master_Log_File: server1-bin.000001
           Read_Master_Log_Pos: 1009
...
         Seconds_Behind_Master: 0
       Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
...
1 row in set (0.001 sec)
```

`Seconds_Behind_Master` 是一个有趣的监控值，因为它可以帮助您查看是否存在复制问题。

### MariaDB 辅助服务器实践坊

对于本实践坊，您需要两台服务器，均已按前面章节的描述安装、配置和保护了 MariaDB 服务。

您将在辅助服务器上配置复制，创建一个新数据库，插入数据，并检查它是否在辅助服务器上可访问。

我们的两台服务器具有以下 IP 地址：

* server1: 192.168.1.100
* server2: 192.168.1.101

请记得将这些值替换为您自己的值。

#### 任务 1：创建专用复制用户

在主服务器上：

```bash
$ sudo mariadb

MariaDB [(none)]> CREATE USER 'replication'@'192.168.1.101' IDENTIFIED BY 'PASSWORD';
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'192.168.1.101';
Query OK, 0 rows affected (0.002 sec)
```

#### 任务 2：记录主服务器值

```bash
$ sudo mariadb

MariaDB [(none)]> FLUSH TABLES WITH READ LOCK;
Query OK, 0 rows affected (0.021 sec)

MariaDB [(none)]> SHOW MASTER STATUS;
+--------------------+----------+--------------+------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------------+----------+--------------+------------------+
| server1-bin.000001 |     1009 |              |                  |
+--------------------+----------+--------------+------------------+
1 row in set (0.000 sec)

MariaDB [(none)]> UNLOCK TABLES;
Query OK, 0 rows affected (0.000 sec)
```

#### 任务 3：激活复制

在辅助服务器上：

```bash
MariaDB [(none)]> CHANGE MASTER TO
  MASTER_HOST='192.168.1.100',
  MASTER_USER='replication',
  MASTER_PASSWORD='PASSWORD',
  MASTER_PORT=3306,
  MASTER_LOG_FILE='server1-bin.000001',
  MASTER_LOG_POS=1009,
  MASTER_CONNECT_RETRY=10;
Query OK, 0 rows affected, 1 warning (0.021 sec)

MariaDB [(none)]> START SLAVE;
Query OK, 0 rows affected (0.001 sec)
```

检查复制是否正常：

```bash
MariaDB [(none)]> SHOW SLAVE STATUS \G
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: 192.168.1.100
                   Master_User: replication
               Master_Log_File: server1-bin.000001
           Read_Master_Log_Pos: 1009
...
         Seconds_Behind_Master: 0
       Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
...
1 row in set (0.001 sec)
```

#### 任务 4：创建新数据库和用户

在主服务器上：

```bash
MariaDB [(none)]> create database NEW_DATABASE_NAME;
Query OK, 1 row affected (0.002 sec)

MariaDB [(none)]> grant all privileges on NEW_DATABASE_NAME.* TO 'NEW_USER_NAME'@'localhost' identified by 'PASSWORD';
Query OK, 0 rows affected (0.004 sec)
```

在辅助服务器上，检查数据库的创建：

```bash
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| NEW_DATABASE_NAME  |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

在辅助服务器上，尝试连接在主服务器上创建的新用户：

```bash
$ mariadb -u NEW_USER_NAME -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| NEW_DATABASE_NAME  |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)
```

#### 任务 5：插入新数据

在主服务器上插入新数据：

```bash
MariaDB [(none)]> use NEW_DATABASE_NAME
Database changed

MariaDB [(none)]>  CREATE TABLE users(
    ->     id INT NOT NULL AUTO_INCREMENT,
    ->     first_name VARCHAR(30) NOT NULL,
    ->     last_name VARCHAR(30) NOT NULL,
    ->     age INT DEFAULT NULL,
    ->     PRIMARY KEY (id));

MariaDB [NEW_DATABASE_NAME]> INSERT INTO users (first_name, last_name, age) VALUES ("Antoine", "Le Morvan", 44);
Query OK, 1 row affected (0.004 sec)

```

在辅助服务器上，检查数据是否已复制：

```bash
MariaDB [(none)]> use NEW_DATABASE_NAME
Database changed

MariaDB [NEW_DATABASE_NAME]> show tables;
+-----------------------------+
| Tables_in_NEW_DATABASE_NAME |
+-----------------------------+
| users                       |
+-----------------------------+
1 row in set (0.000 sec)

MariaDB [NEW_DATABASE_NAME]> SELECT * FROM users;
+----+------------+-----------+------+
| id | first_name | last_name | age  |
+----+------------+-----------+------+
|  1 | Antoine    | Le Morvan |   44 |
+----+------------+-----------+------+
1 row in set (0.000 sec)
```

### MariaDB 辅助服务器知识自测

:heavy_check_mark: 集群中每个服务器必须具有相同的 ID。

* [ ] 正确
* [ ] 错误

:heavy_check_mark: 在激活复制之前必须启用二进制日志。

* [ ] 正确
* [ ] 错误
* [ ] 视情况而定

### MariaDB 辅助服务器总结

如您所见，创建一个或多个辅助服务器是一个相对简单的操作，但它确实需要主服务器上的服务中断。

然而，它提供了许多优势：数据高可用性、负载均衡和简化备份。

在主服务器崩溃时，其中一个辅助服务器可以提升为主服务器。

<!---

## PostgreSQL

In this chapter, you will learn about XXXXXXX.

****

**Objectives**: In this chapter, you will learn how to:

:heavy_check_mark: XXX
:heavy_check_mark: XXX

:checkered_flag: **XXX**, **XXX**

**Knowledge**: :star:
**Complexity**: :star:

**Reading time**: XX minutes

****

### Generalities

### Configuration

### Security

### Workshop

#### Task 1 : XXX

#### Task 2 : XXX

#### Task 3 : XXX

#### Task 4 : XXX

### Check your Knowledge

:heavy_check_mark: Simple question? (3 answers)

:heavy_check_mark: Question with multiple answers?

* [ ] Answer 1
* [ ] Answer 2
* [ ] Answer 3
* [ ] Answer 4

-->
