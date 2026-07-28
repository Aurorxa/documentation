---
title: 在 LAMP 上安装 WordPress
author: Joseph Brinkman
contributors: Steven Spencer, Ganna Zhyrnova 
tested_with: 9.2
---

## 前置条件

- 一个 Rocky Linux 9.x 系统
- sudo 权限

## 简介

WordPress 是一个开源的内容管理系统（CMS），以其[著名的 5 分钟安装](https://developer.wordpress.org/advanced-administration/before-install/howto-install/)而闻名。它通常部署在 LAMP（Linux、Apache、MySQL、PHP）技术栈上。尽管 [XAMPP](https://www.apachefriends.org/)、[Vagrant](https://www.vagrantup.com/) 和 [wp-env](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/) 等高效的本地开发工具广泛可用，但在 LAMP 上手动安装 WordPress 进行本地开发，为寻求更深入理解的初学者提供了宝贵的实践方法。

本指南假设您已经安装了 Rocky Linux 9.x，这涵盖了 LAMP 技术栈的 'L' 部分。

本指南探讨了如何在 Rocky Linux 9 机器上使用 LAMP 技术栈手动安装 WordPress。这不是一份生产就绪的指南，而是一个可以在此基础上构建的起点。本指南中包含的 LAMP 设置，在未首先采取需要额外配置的适当安全措施之前，不建议用于本地开发以外的任何其他用途。

## 升级系统软件包

确保系统的软件包是最新的：

```bash
    sudo dnf upgrade -y
```

## 安装 Apache

Apache 是一个 Web 服务器，将托管您的 WordPress 站点。使用以下命令安装：

```bash
    sudo dnf install httpd -y
```

## 启用 Apache 在启动时启动

安装 Apache 后，启用它以便在启动时自动启动：

```bash
    sudo systemctl enable --now httpd
```

## 安装 MariaDB

WordPress 将动态内容存储在 MySQL 数据库中。MariaDB 是 MySQL 的开源分支。使用以下命令安装：

```bash
    sudo dnf install mariadb-server -y
```

## 启用 MariaDB 服务器

安装 MariaDB 后，启用它以便在启动时自动启动：

```bash
    sudo systemctl enable --now mariadb
```

## 加固 MariaDB

运行 `mysql_secure_installation` 脚本：

```bash
    sudo mysql_secure_installation --use-default
```

此脚本执行以下操作：

1. 如果尚未设置 root 密码，则设置密码

2. 删除匿名用户

3. 禁止远程 root 登录

4. 删除对测试数据库的访问

5. 重新加载权限

## 安装 PHP

PHP 是用于与 MySQL 数据库交互并执行动态操作的编程语言。它在 WordPress 核心、主题和插件中被大量使用。

安装 PHP 和连接 MySQL 所需的包：

```bash
    sudo dnf install php php-mysqlnd php-gd php-xml php-mbstring
```

安装 PHP 后，必须重新加载 Apache 以将其作为 Apache 模块安装并读取其配置文件：

## 重启 Apache

```bash
    sudo systemctl restart httpd
```

## 获取并解压 WordPress

使用 `curl` 下载最新版本的 WordPress：

```bash
    curl -O https://wordpress.org/latest.tar.gz
```

使用 `tar` 解压下载的归档文件：

```bash
    tar -xzvf latest.tar.gz
```

将 WordPress 文件复制到 Apache 的默认公共目录：

```bash
   sudo cp -r wordpress/* /var/www/html 
```

## 设置所有者

让 Apache 成为文件的所有者：

```bash
    sudo chown -R apache:apache /var/www/html/
```

设置 WordPress 文件的权限：

## 设置权限

```bash
    sudo chmod -R 755 /var/www/html/
```

登录到 MySQL CLI：

## 配置数据库

```bash
    sudo mysql -u root -p
```

为您 WordPress 网站创建新数据库：

## 创建新数据库

```bash
    CREATE DATABASE LOCALDEVELOPMENTENV;
```

为您的数据库创建一个带密码的用户：

!!! note

    强烈建议使用更强的密码。

## 创建新用户和密码

```bash
    CREATE USER 'admin'@'localhost' IDENTIFIED BY 'password';
```

将您 WordPress 数据库的所有权限授予刚创建的用户：

```bash
    GRANT ALL PRIVILEGES ON LOCALDEVELOPMENTENV.* TO 'admin'@'localhost';
```

刷新权限以确保更改生效：

```bash
    FLUSH PRIVILEGES;
```

退出 MySQL CLI：

```bash
    EXIT;
```

## 配置 WordPress

复制 `wp-config-sample.php` 模板并重命名：

```bash
    sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```

使用您选择的文本编辑器打开 `wp-config.php` 文件：

```bash
    sudo vi /var/www/html/wp-config.php
```

## 替换数据库设置

您必须在 `wp-config.php` 文件中定义以下常量：

```bash
    define ('DB_NAME', 'LOCALDEVELOPMENTENV');
    define ('DB_USER', 'admin');
    define ('DB_PASSWORD', 'password');
```

## 配置防火墙

在防火墙中打开 HTTP 和 HTTPS 服务：

```bash
    sudo firewall-cmd --add-service=http --add-service=https
```

重新加载 `firewalld` 以确保更改生效：

```bash
    sudo systemctl reload firewalld
```

## SELinux 设置

要允许 Apache 对您的 WordPress 文件进行读写访问，运行此命令：

```bash
   chcon -R -t httpd_sys_rw_content_t /var/www/html/ 
```

要允许 Apache 进行网络连接，运行此命令：

!!! Note

    `-P` 标志使此配置在重启后持久有效

```bash
    setsebool -P httpd_can_network_connect true
```

## 总结

要完成安装，您现在应该可以使用服务器的主机名或私有 IP 地址通过网络连接 WordPress。请记住，此设置主要用于本地开发目的。对于生产使用，您需要配置以下内容：设置域名、安装 SSL 证书、加固 Apache 服务器、微调 SELinux 配置以及实施备份。尽管如此，按照本指南，您已在 LAMP 技术栈上为 WordPress 开发创建了一个坚实的起点。
