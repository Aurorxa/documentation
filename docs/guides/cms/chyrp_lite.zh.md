---
title: Chyrp Lite
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 10.2
tags:
  - cms
  - blogging
---

## 简介

[Chyrp Lite](https://chyrplite.net/) 是一个用 PHP 编写的超轻量级博客引擎。

## 前置条件和假设

以下是使用此过程的最低要求：

* 能够以 root 用户运行命令或使用 `sudo` 提升权限
* 熟悉命令行编辑器。作者在此使用 `vi` 或 `vim`，您可以替换为您喜欢的编辑器

## 安装 Caddy

我们将使用 Caddy 作为 Web 服务器。要安装 Caddy，首先需要安装 EPEL（Enterprise Linux 额外软件包）并运行更新：

```bash
sudo dnf -y install epel-release
```

然后安装 Caddy：

```bash
sudo dnf copr enable @caddy/caddy
```

执行升级以确保系统上有最新的软件包：

```text
sudo dnf upgrade
```

随后，打开 `Caddyfile`：

```bash
vi /etc/caddy/Caddyfile
```

将以下内容添加到您的 `Caddyfile`：

```bash
your.domain.name {
        root * /var/www/chyrp-lite
        file_server
        php_fastcgi 127.0.0.1:9000
}
```

!!! note "对于 `incus` 容器"

    虽然此过程在 `incus` 容器中完全能正常工作，但 `incus` 没有默认启用防火墙。您可以在容器上安装并使用 `firewalld`，只是默认情况下不存在。如果您想将防火墙规则应用到您的容器——如果您使用公共域名，这一点可能非常重要——那么请注意您需要安装：

    ```
    sudo dnf install firewalld
    ``` 

    并启用：

    ```
    sudo systemctl enable --now firewalld
    ```

    在继续此过程之前。

保存文件为 `:wq!`，然后打开相应的防火墙端口：

```bash
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

最后，启动 Caddy：

```bash
systemctl enable --now caddy
```

## 安装 PHP

!!! note

    如果您运行的是 Rocky Linux 8.x 或 9.x，在 Remi 包安装行中将发布版本旁边的数字替换为 "8" 或 "9"。

要安装 PHP，您需要 Remi 仓库。运行以下命令安装 Remi 仓库：

```bash
dnf install https://rpms.remirepo.net/enterprise/remi-release-10.rpm
```

然后安装 PHP 和所需模块：

```bash
dnf install -y php83-php php83-php-session php83-php-json php83-php-ctype php83-php-filter php83-php-libxml php83-php-simplexml php83-php-mbstring php83-php-pdo php83-php-curl
```

接下来，打开 PHP 配置文件：

```bash
vi /etc/opt/remi/php83/php-fpm.d/www.conf
```

向下转到 `listen =` 行并将其设置为以下内容：

```bash
listen = 127.0.0.1:9000
```

使用 `:wq!` 退出 `vi` 并启用 PHP：

```bash
systemctl enable --now php83-php-fpm.service
```

## 安装 Chyrp

首先转到[发布页面](https://github.com/xenocrat/chyrp-lite/releases)安装 chyrp-lite。
通过右键点击 `source.zip` 文件并复制链接来复制最新发布版的 URL。
切换到 `/var/www` 目录：

```bash
cd /var/www
```

将您复制的最新发布版 `source.zip` 文件的 URL 粘贴到 `wget` 中：

```bash
wget [您复制的 URL]
```

接下来，解压并移动提取的文件夹：

```bash
unzip v2026.01.zip
mv chyrp-lite-2026.01/ chyrp-lite
```

设置 `chyrp-lite` 文件夹的正确权限：

```bash
chown -R apache:apache chyrp-lite/
```

设置一个数据目录用于存储 SQLite 数据库：

```bash
mkdir chyrp-lite-data
chown -R apache:apache chyrp-lite-data/
```

对于 `incus` 容器安装，您可以跳过 SELinux 步骤。SELinux 在 `incus` 容器中不存在，也不受支持。

接下来，设置 SELinux 文件上下文：

```bash
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/chyrp-lite(/.*)?"
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/chyrp-lite-data(/.*)?"
restorecon -Rv /var/www/chyrp-lite
restorecon -Rv /var/www/chyrp-lite-data
```

在客户端机器上，打开 Web 浏览器访问 `https://example.com/install.php` 并运行安装程序（将 `example.com` 替换为您的实际域名或主机名）：

![Chyrp Lite Setup](../images/chyrp_lite_setup.png)

在**数据库（Database）**部分，选择之前创建的 `chyrp-lite-data` 目录中的路径名，例如 `/var/www/chyrp-lite-data/sqlite.db`。

然后，填写其他字段，这些字段应该是不言自明的。

接下来，点击**安装我（Install me）**，然后点击**带我去我的网站（Take me to my site）**。您现在应该能够访问一个完整的 Chyrp 站点安装：

![Chyrp Lite](../images/chyrp_lite.png)

## 总结

考虑到 WordPress 已经演变成了 Web 开发的瑞士军刀，一些站长（包括作者在内）更倾向于轻量级的博客引擎也就不足为奇了。Chyrp Lite 非常适合这些用户。
