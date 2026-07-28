---
title: MediaWiki
author: Neel Chauhan
contributors: Steven Spencer
tested_with: 10.0
tags:
  - cms
---

## 简介

[MediaWiki](https://www.mediawiki.org/wiki/MediaWiki) 是一个流行的开源 Wiki 软件引擎，驱动着 Wikipedia、Fandom、wikiHow 等网站。

## 前置条件和假设

以下是使用此过程的最低要求：

* 能够以 root 用户运行命令或使用 `sudo` 提升权限
* 熟悉命令行编辑器。作者在此使用 `vi` 或 `vim`，您可以替换为您喜欢的编辑器

## 安装 Apache

Apache 是您将使用的 Web 服务器。使用以下命令安装：

```bash
dnf -y install httpd
```

接下来，打开相应的防火墙端口：

```bash
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

## 安装 PHP

要安装 PHP，首先需要安装 EPEL（Enterprise Linux 额外软件包）：

```bash
dnf -y install epel-release && dnf -y update
```

您还需要 Remi 仓库。使用以下命令安装：

```bash
dnf install https://rpms.remirepo.net/enterprise/remi-release-10.rpm
```

然后安装 PHP 和所需模块：

```bash
dnf install -y dnf install php84-php-fpm php84-php-intl php84-php-mbstring php84-php-apcu php84-php-curl php84-php-mysql php84-php-xml
```

使用以下命令启用 PHP：

```bash
systemctl enable --now php84-php-fpm.service
```

## 安装 MariaDB

您需要 MariaDB 用于数据库。使用以下命令安装：

```bash
dnf install mariadb-server
```

接下来启用 `systemd` 服务并运行设置向导：

```bash
systemctl enable --now mariadb
mysql_secure_installation
```

当要求输入 root 密码时按 ++enter++ ：

```bash
Enter current password for root (++enter++ for none):
```

对 `unix_socket` 身份验证回答 ++"n"++ ：

```bash
Switch to unix_socket authentication [Y/n] n
```

对更改 root 密码回答 ++"Y"++ 并输入所需的 root 密码：

```bash
Change the root password? [Y/n] Y
New password: 
Re-enter new password: 
```

删除匿名用户并禁止远程 `root` 登录：

```bash
Remove anonymous users? [Y/n] Y
...
Disallow root login remotely? [Y/n] Y
```

删除对测试数据库的访问并重新加载权限表：

```bash
Remove test database and access to it? [Y/n] Y
...
Reload privilege tables now? [Y/n] Y
```

使用以下命令登录 MariaDB：

```bash
mysql -u root -p
```

输入您之前创建的 root 密码。

当您进入 MariaDB 控制台时，创建 MediaWiki 的数据库：

```bash
MariaDB [(none)]> create database mediawiki;
```

接下来，创建 MediaWiki 用户：

```bash
MariaDB [(none)]> create user 'mediawiki'@'localhost' identified by 'nchauhan11';
```

授予对 MediaWiki 数据库的权限：

```bash
grant all privileges on mediawiki.* to 'mediawiki'@'localhost';
```

最后，刷新权限：

```bash
MariaDB [(none)]> flush privileges;
```

## 安装 MediaWiki

转到 `/var/www/` 目录并下载 MediaWiki：

```bash
cd /var/www/
wget https://releases.wikimedia.org/mediawiki/1.44/mediawiki-1.44.0.zip
```

解压并移动 MediaWiki：

```bash
unzip mediawiki-1.44.0.zip
mv mediawiki-1.44.0/* html/
```

设置正确的 SELinux 权限：

```bash
chown -R apache:apache /var/www/html
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html
```

启用 Apache：

```bash
systemctl enable --now httpd
```

接下来，在浏览器中打开 `http://your_ip`（将 `your_ip` 替换为您的 IP 地址）：

![MediaWiki Initial Setup](../images/mediawiki_1.png)

选择您的语言并点击**继续（Continue）**：

![MediaWiki Language Page](../images/mediawiki_2.png)

检查 PHP 配置是否正确，向下滚动并点击**继续（Continue）**：

![MediaWiki PHP Checks](../images/mediawiki_3.png)

现在，按如下方式输入数据库信息：

* **数据库主机（Database host）**：`localhost`

* **数据库名称（Database name，不含连字符）**：`mediawiki`（或在 **MariaDB** 步骤中创建的数据库）

* **数据库用户名（Database username）**：`mediawiki`（或在 **MariaDB** 步骤中创建的用户）

* **数据库密码（Database password）**：您在 **MariaDB** 步骤中创建的密码

![MediaWiki Database Information](../images/mediawiki_4.png)

点击**继续（Continue）**：

![MediaWiki Database Access Settings](../images/mediawiki_5.png)

在 **MediaWiki *版本* 安装**页面中，输入以下内容：

* **URL 主机名（URL host name）**：您想要的 URL

* **Wiki 名称（Name of wiki）**：您想要的 Wiki 名称

* **管理员账户（Administrator account）/您的用户名（Your username）**：您想使用的管理员用户名

* **管理员账户（Administrator account）/密码-再次输入（Password (again)）**：您想使用的管理员密码

* **管理员账户（Administrator account）/电子邮件地址（Email address）**：管理员用户的电子邮件地址

可选地，您也可以选择**多问我一些问题（Ask me more questions）**来微调 Wiki。为简单起见，只需选择**我已经烦了，直接安装 wiki（I'm bored already, just install the wiki）**并点击**继续（Continue）**：

![MediaWiki Wiki Information](../images/mediawiki_6.png)

点击**继续（Continue）**安装 Wiki：

![MediaWiki Install Step Part 1](../images/mediawiki_7.png)

MediaWiki 将设置数据库。完成后，点击**继续（Continue）**：

![MediaWiki Install Step Part 2](../images/mediawiki_8.png)

您的浏览器将下载一个 `LocalSettings.php` 文件。您需要使用 `sftp` 将其上传到服务器。

作为示例，作者将使用他们的 Fedora 42 笔记本电脑上传此文件。操作如下：

```bash
sftp root@your_ip
(输入密码)
cd /var/www/html
put LocalSettings.php 
```

![MediaWiki LocalSettings.php Step](../images/mediawiki_9.png)

最后，点击**进入您的 wiki（enter your wiki）**：

![Fresh MediaWiki Wiki](../images/mediawiki_10.png)

您现在有了一个全新的 MediaWiki 安装。

## 总结

虽然 MediaWiki 最广为人知的是为 Wikipedia 提供动力，但它在用户需要编辑页面能力的内容管理系统中也非常有用。MediaWiki 是 Microsoft SharePoint 的一个优秀的开源替代方案。
