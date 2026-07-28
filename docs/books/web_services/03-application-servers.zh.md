---
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
title: 第 3 部分. 应用服务器
tags:
  - web
  - php
  - php-fpm
  - application server
  - dynamic language
--- 

## PHP 和 PHP-FPM

在本章中，您将学习 PHP 和 PHP-FPM 的相关知识。

**PHP**（**P**HP **H**ypertext **P**reprocessor，PHP 超文本预处理器）是一种专门为 Web 应用开发设计的脚本语言。2024 年，PHP 在全球生成的 Web 页面中占比略低于 80%。PHP 是开源的，是最著名的 CMS（WordPress、Drupal、Joomla!、Magento 等）的核心。

**PHP-FPM**（**F**astCGI **P**rocess **M**anager，FastCGI 进程管理器）自 5.3.3 版本起已集成到 PHP 中。FastCGI 版本的 PHP 带来了额外的功能。

****

**目标**：您将学习如何：

:heavy_check_mark: 安装 PHP 应用服务器  
:heavy_check_mark: 配置 PHP-FPM 池  
:heavy_check_mark: 优化 PHP-FPM 应用服务器  

:checkered_flag: **PHP**, **PHP-FPM**, **Application server**

**知识**：:star: :star: :star:  
**复杂性**：:star: :star: :star:

**阅读时间**：30 分钟

****

### 概述

**CGI**（**C**ommon **G**ateway **I**nterface，通用网关接口）和 **FastCGI**（快速通用网关接口）允许 Web 服务器（Apache 或 Nginx）和开发语言（PHP、Python、Java）之间进行通信：

* 在 **CGI** 的情况下，每个请求创建一个**新进程**，性能效率较低。
* **FastCGI** 依赖**一定数量的进程**来处理其客户端请求。

PHP-FPM，**除了更好的性能外**，还带来了：

* 更好地**分区应用程序**的可能性：以不同的 uid/gid 启动进程，使用个性化的 `php.ini` 文件，
* 统计信息管理，
* 日志管理，
* 进程的动态管理和不停服务重启（'graceful'）。

!!! Note

    由于 Apache 有内置的 PHP 模块，php-fpm 更常用于 Nginx 服务器。

### PHP 版本

在 Rocky Linux 10 中，与其上游一样，没有 module（模块）。这意味着当您安装 PHP 时，您将从 Appstream 仓库获取包。要发现是什么版本，请使用此命令：

```bash
dnf whatprovides php
Last metadata expiration check: 0:03:22 ago on Tue 21 Oct 2025 02:40:23 PM UTC.
php-8.3.19-1.el10_0.x86_64 : PHP scripting language for creating dynamic web sites
Repo        : appstream
Matched from:
Provide    : php = 8.3.19-1.el10_0
```

如果您使用的是更新版本的 10，您的版本可能有所不同。

### 安装 PHP CGI 模式

首先，安装并使用 CGI 模式的 PHP。它只能与 Apache Web 服务器及其 `mod_php` 模块配合使用。本文档的 FastCGI 部分（php-fpm）解释了如何将 PHP 与 Nginx（以及 Apache）集成。

安装 PHP 相对简单。它包括安装主包和您需要的几个模块。

以下示例安装了通常包含的模块的 PHP。

 ```bash
 sudo dnf install php php-cli php-gd php-curl php-zip php-mbstring
 ```

使用以下命令检查您的版本：

```bash
php -v
PHP 8.3.19 (cli) (built: Mar 12 2025 13:10:27) (NTS gcc x86_64)
Copyright (c) The PHP Group
Zend Engine v4.3.19, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.19, Copyright (c), by Zend Technologies
```

### Apache 集成

要使用 CGI 模式提供 PHP 页面，您必须安装并配置 Apache 服务器，激活并启动它。

* 安装：

 ```bash
 sudo dnf install httpd
 ```

    激活：

 ```bash
 sudo systemctl enable --now httpd
 sudo systemctl status httpd
 ```

* 不要忘记配置防火墙：

 ```bash
 sudo firewall-cmd --add-service=http --permanent
 sudo firewall-cmd --reload
 ```

默认 vhost 应该开箱即用。PHP 提供了 `phpinfo()` 函数，该函数生成其配置的摘要表。这对于测试 PHP 是否正常工作很有帮助。但是，请不要将这些测试文件留在您的服务器上。它们对您的基础设施构成重大的安全风险。

创建文件 `/var/www/html/info.php`（`/var/www/html` 是默认 Apache 配置的默认 vhost 目录）：

```bash
<?php
phpinfo();
?>
```

使用 Web 浏览器访问 [http://your-server-ip/info.php](http://your-server-ip/info.php) 来验证服务器是否正常工作。

!!! Warning

    不要将 `info.php` 文件留在您的服务器上！

### 安装 PHP CGI 模式（PHP-FPM）

如前所述，切换到 PHP-FPM 进行 Web 托管有许多优势。

安装只需要 php-fpm 包：

```bash
sudo dnf install php-fpm
```

由于 php-fpm 从系统角度看是一个服务，您必须激活并启动它：

```bash
sudo systemctl enable --now php-fpm
sudo systemctl status php-fpm
```

#### PHP CGI 模式配置

主配置文件是 `/etc/php-fpm.conf`。

```bash
include=/etc/php-fpm.d/*.conf
[global]
pid = /run/php-fpm/php-fpm.pid
error_log = /var/log/php-fpm/error.log
daemonize = yes
```

!!! Note

    php-fpm 配置文件的注释非常详细。去看一看吧！

正如您所见，`/etc/php-fpm.d/` 目录中扩展名为 `.conf` 的文件始终会被包含。

默认情况下，一个名为 `www` 的 PHP 进程池声明在 `/etc/php-fpm.d/www.conf` 中。

```bash
[www]
user = apache
group = apache

listen = /run/php-fpm/www.sock
listen.acl_users = apache,nginx
listen.allowed_clients = 127.0.0.1

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35

slowlog = /var/log/php-fpm/www-slow.log

php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache
```

| 指令 | 描述                                                   |
|--------------|---------------------------------------------------------------|
| `[pool]`     | 进程池名称。配置文件可以包含多个进程池（括号中的池名称开始一个新部分）。 |
| `listen`     | 定义监听接口或使用的 Unix socket。 |

#### 配置访问 php-fpm 进程的方式

有两种连接方式。

使用 `inet-interface`（网络接口），例如：

`listen = 127.0.0.1:9000`。

或使用 UNIX socket（Unix 域套接字）：

`listen = /run/php-fpm/www.sock`。

!!! Note

    当 Web 服务器和 PHP 服务器在同一台机器上时，使用 socket 会移除 TCP/IP 层，从而优化性能。

当使用接口工作时，您必须配置 `listen.owner`、`listen.group`、`listen.mode` 来指定 UNIX socket 的所有者、所有者组和权限。**警告：** 两个服务器（Web 和 PHP）都必须具有 socket 的访问权限。

当使用 socket 工作时，您必须配置 `listen.allowed_clients` 来限制对 PHP 服务器的访问仅限特定的 IP 地址。

示例：`listen.allowed_clients = 127.0.0.1`

#### 静态或动态配置

您可以静态或动态地管理 PHP-FPM 进程。

在静态模式中，`pm.max_children` 设置子进程数量的限制：

```bash
pm = static
pm.max_children = 10
```

此配置以 10 个进程启动。

在动态模式中，PHP-FPM 最多启动 `pm.max_children` 值指定的进程数。它首先启动一些对应于 `pm.start_servers` 的进程，至少保持 `pm.min_spare_servers` 值的空闲进程，最多保持 `pm.max_spare_servers` 的空闲进程。

示例：

```bash
pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
```

PHP-FPM 将创建一个新进程来替换已处理 `pm.max_requests` 次请求的进程。

默认情况下，`pm.max_requests` 设置为 0，意味着进程永远不会被回收。`pm.max_requests` 选项对于存在内存泄漏的应用程序可能很有吸引力。

第三种运行模式是 `ondemand` 模式。此模式仅在收到请求时启动进程。对于具有强大影响力的站点来说，这不是最优模式，而是为特定需求保留的（请求微弱的站点、管理后台等）。

!!! Note

    PHP-FPM 运行模式配置对确保 Web 服务器的最佳运行至关重要。

#### 进程状态

像 Apache 及其 `mod_status` 模块一样，PHP-FPM 提供了一个显示进程状态的页面。

要激活该页面，使用 `pm.status_path` 指令设置其访问路径：

```bash
pm.status_path = /status
```

```bash
$ curl http://localhost/status_php
pool:                 www
process manager:      dynamic
start time:           03/Dec/2021:14:00:00 +0100
start since:          600
accepted conn:        548
listen queue:         0
max listen queue:     15
listen queue len:     128
idle processes:       3
active processes:     3
total processes:      5
max active processes: 5
max children reached: 0
slow requests:        0
```

#### 记录过长请求

`slowlog` 指令指定接收过长请求日志的文件（例如，时间超过 `request_slowlog_timeout` 指令值的请求）。

生成文件的默认位置是 `/var/log/php-fpm/www-slow.log`。

```bash
request_slowlog_timeout = 5
slowlog = /var/log/php-fpm/www-slow.log
```

`request_slowlog_timeout` 值为 0 会禁用日志记录。

### Nginx 集成

Nginx 的默认设置已经包含了使 PHP 与 PHP-FPM 工作所需的配置。

配置文件 `fastcgi.conf`（或 `fastcgi_params`）位于 `/etc/nginx/` 下：

```bash
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# 仅 PHP，如果 PHP 使用 --enable-force-cgi-redirect 构建则需要
fastcgi_param  REDIRECT_STATUS    200;
```

要让 Nginx 处理 `.php` 文件，将以下指令添加到站点配置文件：

如果 PHP-FPM 监听在 9000 端口：

```bash
location ~ \.php$ {
  include /etc/nginx/fastcgi_params;
  fastcgi_pass 127.0.0.1:9000;
}
```

如果 php-fpm 监听在 UNIX socket：

```bash
location ~ \.php$ {
  include /etc/nginx/fastcgi_params;
  fastcgi_pass unix:/run/php-fpm/www.sock;
}
```

### Apache 集成

配置 Apache 使用 PHP 池非常简单。您必须使用代理模块并配合 `ProxyPassMatch` 指令，例如：

```bash
<VirtualHost *:80>
  ServerName web.rockylinux.org
  DocumentRoot "/var/www/html/current/public"

  <Directory "/var/www/html/current/public">
    AllowOverride All
    Options -Indexes +FollowSymLinks
    Require all granted
  </Directory>
  ProxyPassMatch ^/(.*\.php(/.*)?)$ "fcgi://127.0.0.1:9000/var/www/html/current/public"

</VirtualHost>

```

### PHP 池的可靠配置

优化服务的请求数量和 PHP 脚本使用的内存分析对于最大化启动线程数是必要的。

首先，您需要通过以下命令了解 PHP 进程的平均内存使用量：

```bash
while true; do ps --no-headers -o "rss,cmd" -C php-fpm | grep "pool www" | awk '{ sum+=$1 } END { printf ("%d%s\n", sum/NR/1024,"Mb") }' >> avg_php_proc; sleep 60; done
```

这将为您提供该服务器上 PHP 进程平均内存占用的相当准确的概念。

本文档的后续部分将给出在满负载下每个进程 120MB 的内存占用。

在一台具有 8Gb RAM 的服务器上，保留 1Gb 给系统和 1Gb 给 OPCache（参见本文档的后续部分），剩余 6Gb 用于处理来自客户端的 PHP 请求。

您可以得出结论，此服务器最多可以接受 **50 个线程** `((6*1024) / 120)`。

针对此用例的 `php-fpm` 示例配置：

```bash
pm = dynamic
pm.max_children = 50
pm.start_servers = 12
pm.min_spare_servers = 12
pm.max_spare_servers = 36
pm.max_requests = 500
```

其中：

* `pm.start_servers` = `max_children` 的 25%
* `pm.min_spare_servers` = `max_children` 的 25%
* `pm.max_spare_servers` = `max_children` 的 75%

### Opcache 配置

`opcache`（Optimizer Plus Cache，优化器增强缓存）是您可以影响的第一级缓存。

它将已编译的 PHP 脚本保留在内存中，这对 Web 页面的执行有强烈影响（消除了从磁盘读取脚本 + 编译的时间）。

要配置它，您必须处理：

* 根据命中率正确配置分配给 opcache 的内存大小
* 要缓存的 PHP 脚本数量（键的数量 + 最大脚本数）
* 要缓存的字符串数量

安装它：

```bash
sudo dnf install php-opcache
```

要配置它，编辑 `/etc/php.d/10-opcache.ini` 配置文件：

```bash
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
```

其中：

* `opcache.memory_consumption` 对应于 opcache 所需的内存量（增加此值直到获得正确的命中率）。
* `opcache.interned_strings_buffer` 是要缓存的字符串数量。
* `opcache.max_accelerated_files` 接近 `find ./ -iname "*.php"|wc -l` 命令的结果。

配置 opcache 时，请参考一个 `info.php` 页面（包含 `phpinfo();`）（例如，查看 `Cached scripts` 和 `Cached strings` 的值）。

!!! Note

    在每次部署新代码时，都需要清空 opcache（例如通过重启 php-fpm 进程）。

!!! Note

    不要低估正确设置和配置 opcache 可以获得的性能提升。

<!---

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

## Python

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
