---
title: 使用 Nextcloud 搭建云服务器
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - cloud
  - nextcloud
---


## 前置条件和假设

- 运行 Rocky Linux 的服务器（您可以在任何 Linux 发行版上安装 Nextcloud，但此过程假设您使用的是 Rocky）。
- 能够舒适地从命令行操作进行安装和配置。
- 了解命令行编辑器。此处使用 `vi` 作为示例，但您可以使用您喜欢的编辑器。
- 此过程涵盖 `.zip` 文件安装方法。您也可以通过 snap 安装 Nextcloud。
- 此过程使用 Apache _sites enabled_ 文档（稍后链接）进行目录设置。
- 此过程还使用 _mariadb-server_ 加固（也稍后链接）进行数据库设置。
- 贯穿本文档，假设您是 root，或者可以通过 `sudo` 提升权限。
- 此处使用的示例域名是 "yourdomain.com"。

## 简介

如果您负责大型（甚至小型）公司的服务器环境，请考虑使用云应用程序。在云端处理事务可以释放您自己的资源用于其他任务，但有一个缺点：失去对公司数据的控制。如果云应用程序遭受入侵，您的公司数据也可能面临风险。

将云端带回自己的环境是重新获得对数据控制的一种方式，但这需要付出您的时间和精力。有时，这是值得付出的代价。

Nextcloud 提供了一个开源的云平台，以安全性和灵活性为重点。请注意，搭建 Nextcloud 服务器是一个很好的练习，即使您最终将云放在外部也是如此。以下过程涉及在 Rocky Linux 上设置 Nextcloud。

## Nextcloud 安装

### 安装和配置仓库和模块

对于此安装，您需要两个仓库。需要安装 EPEL（Enterprise Linux 额外软件包）和用于版本 10 的 Remi 仓库。

!!! note

    虽然 Rocky Linux 10 需要 PHP 8.3，但 Remi 仓库提供了 Nextcloud 所需的额外 PHP 包。

要安装 EPEL，运行：

```bash
dnf install epel-release
```

要安装 Remi 仓库，运行：

```bash
dnf install https://rpms.remirepo.net/enterprise/remi-release-10.rpm
```

然后再次运行 `dnf upgrade`。

运行以下命令查看可用的 PHP 模块列表：

```bash
dnf module list php
```

对于 Rocky Linux 10，这将给出以下输出：

```bash
Remi's Modular repository for Enterprise Linux 10 - x86_64
Name                   Stream                      Profiles                                      Summary                                  
php                    remi-7.4                    common [d], devel, minimal                    PHP scripting language                   
php                    remi-8.0                    common [d], devel, minimal                    PHP scripting language                   
php                    remi-8.1                    common [d], devel, minimal                    PHP scripting language                   
php                    remi-8.2                    common [d], devel, minimal                    PHP scripting language                   
php                    remi-8.3                    common [d], devel, minimal                    PHP scripting language                   
php                    remi-8.4                    common [d], devel, minimal                    PHP scripting language

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

使用 Nextcloud 兼容的最新 PHP。目前，这是 8.4。使用以下命令启用该模块：

```bash
dnf module enable php:remi-8.4
```

要查看这如何更改模块列表输出，再次运行模块列表命令，您将在 8.4 旁边看到 "[e]"：

```bash
dnf module list php
```

输出相同，除了这一行：

```bash
php                    remi-8.4 [e]                   common [d], devel, minimal                  PHP scripting language
```

### 安装软件包

此处示例使用 Apache 和 MariaDB。安装所需软件包，请执行以下操作：

```bash
dnf install httpd mariadb-server vim wget zip unzip libxml2 openssl php84-php php84-php-ctype php84-php-curl php84-php-gd php84-php-iconv php84-php-json php84-php-libxml php84-php-mbstring php84-php-openssl php84-php-posix php84-php-session php84-php-xml php84-php-zip php84-php-zlib php84-php-pdo php84-php-mysqlnd php84-php-intl php84-php-bcmath php84-php-gmp
```

### 配置

#### 配置 Apache

设置 _apache_ 在启动时启动：

```bash
systemctl enable httpd
```

然后启动它：

```bash
systemctl start httpd
```

#### 创建配置

在_前置条件和假设_部分中，曾说明您将使用 [Apache Sites Enabled](../web/apache-sites-enabled.md) 过程进行配置。点击该过程设置基础知识，然后返回本文档继续。

对于 Nextcloud，您需要创建以下配置文件：

```bash
vi /etc/httpd/sites-available/com.yourdomain.nextcloud
```

内容如下：

```bash
<VirtualHost *:80>
  DocumentRoot /var/www/sub-domains/com.yourdomain.nextcloud/html/
  ServerName  nextcloud.yourdomain.com
  <Directory /var/www/sub-domains/com.yourdomain.nextcloud/html/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

完成后，保存更改（对于 _vi_，使用 ++shift+colon+"w"+"q"+exclam++）。

接下来，在 `/etc/httpd/sites-enabled` 中创建此文件的链接：

```bash
ln -s /etc/httpd/sites-available/com.yourdomain.nextcloud /etc/httpd/sites-enabled/
```

#### 创建目录

如前面配置中所述，您需要创建 _DocumentRoot_。使用以下命令：

```bash
mkdir -p /var/www/sub-domains/com.yourdomain.com/html
```

这就是您将安装 Nextcloud 实例的位置。

#### 配置 PHP

您需要设置 PHP 的时区。为此，使用您喜欢的文本编辑器打开 `php.ini`：

```bash
vi /etc/opt/remi/php84/php.ini
```

然后找到以下行：

```php
;date.timezone =
```

移除注释（++分号++）并设置时区。对于此示例时区，您可以输入以下任一内容：

```php
date.timezone = "America/Chicago"
```

或

```php
date.timezone = "US/Central"
```

然后保存并退出 `php.ini` 文件。

请注意，为保持一致性，`php.ini` 文件中的时区应与您机器的时区设置相匹配。您可以通过以下方式发现机器的时区：

```bash
ls -al /etc/localtime
```

假设您在安装 Rocky Linux 时设置了时区并且处于 Central 时区，您应该看到类似以下内容：

```bash
/etc/localtime -> /usr/share/zoneinfo/America/Chicago
```

#### 配置 mariadb-server

设置 `mariadb-server` 在启动时启动：

```bash
systemctl enable mariadb
```

然后启动它：

```bash
systemctl restart mariadb
```

同样，如前所述，使用 [加固 `mariadb-server`](../database/database_mariadb-server.md) 的设置过程进行初始配置。

### 安装 .zip

接下来的几个步骤假设您通过 `ssh` 远程连接到 Nextcloud 服务器，并打开了远程控制台：

- 导航到 [Nextcloud 网站](https://nextcloud.com/)。
- 将鼠标悬停在 'Download' 上，将打开下拉菜单。
- 点击 'Nextcloud server'。
- 点击 'Download server archive'。
- 右键点击 'Get ZIP file' 并复制链接。
- 在 Nextcloud 服务器的远程控制台中，输入 `wget`，然后空一格，并粘贴您刚才复制的内容。您应该得到类似如下内容：`wget https://download.nextcloud.com/server/releases/latest.zip`。
- 按 enter 后，.zip 文件的下载将快速开始并完成。

下载完成后，通过以下命令解压 Nextcloud 的 .zip 文件：

```bash
unzip latest.zip
```

### 复制内容并更改权限

完成 .zip 文件解压步骤后，您现在应该在 `/root` 中有一个名为 "nextcloud" 的新目录。进入此目录：

```bash
cd nextcloud
```

将内容复制或移动到 _DocumentRoot_：

```bash
cp -Rf * /var/www/sub-domains/com.yourdomain.nextcloud/html/
```

或

```bash
mv * /var/www/sub-domains/com.yourdomain.nextcloud/html/
```

下一步是确保 Apache 拥有该目录。使用以下命令：

```bash
chown -Rf apache.apache /var/www/sub-domains/com.yourdomain.nextcloud/html
```

出于安全原因，您还需要将 _data_ 文件夹从 _DocumentRoot_ 内部移到外部。使用以下命令：

```bash
mv /var/www/sub-domains/com.yourdomain.nextcloud/html/data /var/www/sub-domains/com.yourdomain.nextcloud/
```

### 配置 Nextcloud

确保您的服务正在运行。如果您遵循了前面的步骤，它们应该已经在运行。由于在初始服务启动之后经历了好几个步骤，最好重启它们以确保：

```bash
systemctl restart httpd
systemctl restart mariadb
```

如果一切重新启动且没有问题，那么您就可以继续了。

要进行初始配置，您需要在 Web 浏览器中加载该站点：

<http://your-server-hostname/>（替换为您的实际主机名）

假设到目前为止一切正确，您应该看到一个 Nextcloud 设置屏幕：

![nextcloud login screen](../images/nextcloud_screen.jpg)

有几件事您需要与默认设置不同：

- 在网页顶部，写着 `Create an admin account` 的地方，设置用户和密码。以本示例为例，输入 `admin` 并设置一个强密码。请务必将此密码保存在安全的地方（如密码管理器），以免丢失。即使您已在此字段中输入，==在完成**所有**字段之前，请勿按== ++enter++。
- 在 `Storage & database` 部分，将 `Data folder` 位置从默认文档根目录更改为您之前移动数据文件夹的位置：`/var/www/sub-domains/com.yourdomain.nextcloud/data`。
- 在 `Configure the database` 部分，点击该按钮将 `SQLite` 更改为 `MySQL/MariaDB`。
- 将您之前设置的 MariaDB root 用户和密码输入到 `Database user` 和 `Database password` 字段。
- 在 `Database name` 字段中输入 `nextcloud`。
- 在 `localhost` 字段中输入 <localhost:3306>（3306 是默认的 _mariadb_ 连接端口）。

完成所有这些后，点击 `Finish Setup`，您应该就可以正常使用了。

浏览器窗口将刷新片刻，然后通常不会重新加载站点。在浏览器窗口中再次输入您的 URL，您应该会看到默认首页。

您的管理员用户已经登录（或应该已经登录），并且有几个信息页面帮助您快速上手。"Dashboard" 是用户首次登录时会看到的界面。管理员用户现在可以创建其他用户、安装其他应用程序以及执行许多不同的任务。

"Nextcloud Manual.pdf" 文件是用户手册，以便用户可以熟悉可用功能。管理员用户应该阅读或至少浏览 [Nextcloud 网站上的管理员手册](https://docs.nextcloud.com/server/21/admin_manual/) 重点内容。

## 后续步骤

至此，不要忘记这是一个将存储公司数据的服务器。重要的是要使用防火墙加以保护，设置[备份](../backup/rsnapshot_backup.md)，使用 [SSL](../security/generating_ssl_keys_lets_encrypt.md) 保护站点，并完成其他必要的任务以保护您的数据安全。

## 结论

您需要仔细评估将公司的云在内部部署的任何决策。对于那些更愿意将公司数据保留在本地而不是使用外部云托管商的人来说，Nextcloud 是一个很好的选择。
