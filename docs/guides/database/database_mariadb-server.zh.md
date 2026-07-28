---
title: 数据库服务器 MariaDB
author: Steven Spencer, Franco Colussi
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - database
  - mariadb
---

# MariaDB 数据库服务器

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够通过命令行文本编辑器编辑配置文件（使用 `vi`，但可替换为您熟悉的编辑器）
* 了解 `mysql` 数据库的安装、执行 SQL 查询的命令行，以及基本的修复/备份/优化知识。

## 引言

Rocky Linux 提供了许多开源工具用于搭建网站，许多用户选择使用 *LAMP* 技术栈，其中包括 *Linux*、*Apache*、*MariaDB* 和 *PHP*。许多流行的 Web 应用程序，例如 WordPress，都依赖 MariaDB/Mysql 数据库来存储数据。如果您运行的网站同时有大量并发用户，那么很容易出现数据库瓶颈。(通常)可以通过服务器配置来缓解这个问题。

如果您使用命令行编辑器进行更改，确保在 `/etc/my.cnf.d/mariadb-server.cnf` 中调整 MariaDB 服务器设置（通过手动编辑文件）。但是，如果修改配置文件不是您喜欢的管理方式，MariaDB 会在重启时读取位于 `/etc/my.cnf.d/` 目录下的 `.cnf` 文件。

通常，MariaDB 的主配置文件是 `/etc/my.cnf.d/mariadb-server.cnf`。此外，除了包含来自该目录的其他配置文件外，还有一个全局服务器配置文件，路径为 `/etc/my.cnf`。

您会注意到，数据库在更新系统后也可能需要微调。如果出现性能问题，请检查 MariaDB 服务器配置文件。

本文档仅涵盖 `mariadb-server.cnf`。如果您对变量调整或 MariaDB 调优的完整文档感兴趣，互联网上有很多深入的资料。此页面并没有太大不同；它只是提供了一些有帮助的入门点。

## `mariadb-server.cnf` —— 简单入门

以下是 MariaDB 安装后，`/etc/my.cnf.d/mariadb-server.cnf` 文件的最低配置样子。

```bash
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mariadb/mariadb.log
pid-file=/run/mariadb/mariadb.pid
```

以下是您可以调整来提升性能的条目。前五个是跨平台的（适用于任何系统），而我们建议使用 `innodb_log_file_size = 512M` 到 `innodb_log_file_size = 1G` 来支持 InnoDB 存储。

### 跨平台设置

* `innodb_buffer_pool_size` - 强烈建议将此值设置为 RAM 总容量的 50% 到 66%。这是我们建议您首先调整的选项。例如，如果您的服务器有 4GB 的 RAM，设置为 `2G`，如果服务器有 32GB，设置为 `20G`。请注意，`innodb_buffer_pool_size` 取决于 `innodb_log_file_size`，允许日志文件大小（`innodb_log_file_size`）最大可达 `innodb_buffer_pool_size` 的 25%。
* `innodb_buffer_pool_instances` - 通常设置为 `innodb_buffer_pool_size` 除以 2 的值再转换为 GB 的结果。当然并不是唯一解答，但这是个不错的起点。
* `innodb_log_file_size` - 通常介于 `512M` 和 `1G` 之间。这决定了在 `innodb_buffer_pool` 中刷新脏页之前可以写入多少日志信息。较大的日志文件意味着服务器需要更少的磁盘写入，这可以加快速度，但也会导致重启后恢复时间过长。作者通常将其设置为 `768M`，这是一个不错的中间值。同样，您需要测试以确定什么适合您的环境。
* `join_buffer_size` - 为没有索引的表连接设置缓冲区最小值。默认值仅为 `256k`。将其设置为 `2M` 可以修复许多无索引连接，但您也可能需要为此类连接添加索引。
* `max_connections` - 如果您有高流量服务器，您可能需要对服务器配置进行调整以支持更高的连接数。默认值 `151` 在许多情况下可能不够。

### Rocky Linux 特定设置（Red Hat 系列）

以下选项在 Rocky Linux（以及其他基于 RHEL 的 Linux 发行版）上可用：

* `innodb_io_capacity` - 在 Rocky Linux 8 中应设置为 1000，在 Rocky Linux 9 中应设置为 800。这些数字是特定于底层硬件的。请参阅下方链接中的 MariaDB 知识库内容。

### 示例

```text
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mariadb/mariadb.log
pid-file=/run/mariadb/mariadb.pid
innodb_buffer_pool_size=2G
innodb_buffer_pool_instances=1
innodb_log_file_size=768M
innodb_io_capacity=1000
join_buffer_size=2M
max_connections=300
```

## 修改配置后

对 `mariadb-server.cnf` 进行任何更改后，您需要重启 MariaDB 服务：

```bash
systemctl restart mariadb
```

如果您的服务器已经是高流量的，且您有除了 MariaDB 之外的其他运行服务（例如 Apache），您需要计划好进行这次重启，因为它会导致所有数据库表暂时不可用，这对您的网站可能是致命的。

## 结论

虽然这只是整体性能调优的某方面，但对 MariaDB 数据库服务器的修改可以让您摆脱网站性能瓶颈。互联网上有许多有关这些变量的详细文章。请记得对任何更改进行测试，以确保它们对您的环境有效。

如果您不熟悉这些数据库概念，以下参考文献可以帮到您。

* [MySQL/MariaDB `innodb`](https://mariadb.com/docs/server/storage-engines/innodb/)
* [MariaDB `innodb_io_capacity`](https://mariadb.com/docs/server/ref/mdb/system-variables/innodb_io_capacity/)
* [MariaDB 系统变量完整列表](https://mariadb.com/kb/en/server-system-variables/#innodb_io_capacity)
