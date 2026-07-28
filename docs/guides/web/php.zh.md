---
title: PHP 与 PHP-FPM
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova, Joseph Brinkman
tested_with: 10.0 
tags:
  - web
  - php
  - php-fpm
---

# PHP 与 PHP-FPM

**PHP** (**P**HP **H**ypertext **P**reprocessor) 是一种专为 Web 应用开发而设计的源脚本语言。在 2024 年，PHP 占据了全球生成网页的近 80%。PHP 是开源的，是最著名的 CMS 的核心（WordPress、Drupal、Joomla!、Magento 等）。

PHP 自 5.3.3 版本起集成了 **PHP-FPM** (**F**astCGI **P**rocess **M**anager)。FastCGI 版本的 PHP 带来了额外的功能。

## 概述

**CGI** (**C**ommon **G**ateway **I**nterface) 和 **FastCGI** 允许 Web 服务器（Apache、Nginx 等）与开发语言（PHP、Python、Java）之间的通信：

* 在 **CGI** 的情况下，每个请求创建一个 **新进程**，这在性能上效率较低。
* **FastCGI** 依赖于 **一定数量的进程** 来处理其客户端请求。

PHP-FPM，**除了性能更好之外**，还带来了：

* 更好地 **分区应用程序** 的可能性：以不同的 uid/gid 启动进程，使用个性化的 `php.ini` 文件，
* 统计信息管理，
* 日志管理，
* 进程的动态管理和无需服务中断的重启（'graceful'/平滑）。

!!! Note

    由于 Apache 有 PHP 模块，php-fpm 在 Nginx 服务器上更常用。

## 选择 PHP 版本

Rocky Linux 10 与其上游一样，不再有模块（modules）。PHP 的版本就是 Appstream 仓库中可用的版本。要检查可用的 PHP 版本，使用：

```bash
dnf whatprovides php

Last metadata expiration check: 0:00:03 ago on Wed 22 Oct 2025 03:58:30 PM UTC.

php-8.3.19-1.el10_0.x86_64 : PHP scripting language for creating dynamic web sites
Repo        : @System
Matched from:
Provide    : php = 8.3.19-1.el10_0

php-8.3.19-1.el10_0.x86_64 : PHP scripting language for creating dynamic web sites
Repo        : appstream
Matched from:
Provide    : php = 8.3.19-1.el10_0
```

现在你可以继续安装 PHP 引擎了。

## PHP CGI 模式

首先，安装 PHP。你只能让它与 Apache Web 服务器及其 `mod_php` 模块一起工作。稍后你将在本文档的 FastCGI 部分 (`php-fpm`) 中看到如何将 PHP 集成到 Nginx 和 Apache 中。

### 安装

PHP 的安装相对简单，因为它包括安装主软件包以及你需要的几个模块。

此示例安装 PHP 以及通常与之一同安装的模块：

```bash
sudo dnf install php php-cli php-gd php-curl php-zip php-mbstring
```

你可以检查安装的版本是否与预期版本一致：

```bash
php -v
PHP 8.3.19 (cli) (built: Mar 12 2025 13:10:27) (NTS gcc x86_64)
Copyright (c) The PHP Group
Zend Engine v4.3.19, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.19, Copyright (c), by Zend Technologies
```

### 配置

#### Apache 集成

要以 CGI 模式提供 PHP 页面，你必须安装 Apache 服务器，配置它，激活它，并启动它。

* 安装：

 ```bash
 sudo dnf install httpd
 ```

* 激活：

 ```bash
 sudo systemctl enable --now httpd
 sudo systemctl status httpd
 ```

* 不要忘记配置防火墙：

 ```bash
 sudo firewall-cmd --add-service=http --permanent
 sudo firewall-cmd --reload
 ```

默认的 vhost (虚拟主机) 应该立即生效。PHP 提供了一个 `phpinfo()` 函数，该函数生成其配置的摘要表格。它对于测试 PHP 的良好工作非常有用。但是，注意不要在你的服务器上留下这样的测试文件。它们对你的基础设施构成巨大的安全风险。

创建文件 `/var/www/html/info.php`（`/var/www/html` 是默认 Apache 配置的默认 vhost 目录）：

```bash
<?php
phpinfo();
?>
```

使用 Web 浏览器前往页面 [http://your-server-ip/info.php](http://your-server-ip/info.php) 检查服务器是否正常工作。

!!! Warning

    不要将 `info.php` 文件留在你的服务器上！

## PHP-FPM (FastCGI)

正如我们在本文档前面所强调的，将 Web 托管切换到 PHP-FPM 模式有很多优势。

### 安装 `php-fpm`

要安装 `php-fpm` 软件包，使用：

```bash
sudo dnf install php-fpm
```

由于 `php-fpm` 从系统角度看是一个服务，你必须激活并启动它：

```bash
sudo systemctl enable --now php-fpm
sudo systemctl status php-fpm
```

### 配置

`php-fpm` 将主配置文件存储在 `/etc/php-fpm.conf` 下：

```bash
include=/etc/php-fpm.d/*.conf
[global]
pid = /run/php-fpm/php-fpm.pid
error_log = /var/log/php-fpm/error.log
daemonize = yes
```

!!! Note

    `php-fpm` 配置文件有大量注释。去看看！

如你所见，`/etc/php-fpm.d/` 目录中带有 `.conf` 扩展名的文件总是会被包含。

默认情况下，`php-fpm` 在 `/etc/php-fpm.d/www.conf` 中声明了一个名为 `www` 的 PHP 进程池：

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
| `[pool]`     | 进程池名称。配置文件可以包含多个进程池（方括号中的池名称开始一个新区段）。 |
| `listen`     | 定义监听接口或使用的 Unix 套接字。 |

#### 配置访问 `php-fpm` 进程的方式

有两种连接方式。

使用 inet 接口，例如：

`listen = 127.0.0.1:9000`。

或使用 Unix 套接字：

`listen = /run/php-fpm/www.sock`。

!!! Note

    当 Web 服务器和 PHP 服务器在同一台机器上时，使用套接字可以移除 TCP/IP 层并优化性能。

使用接口时，你必须配置 `listen.owner`、`listen.group`、`listen.mode` 来指定 Unix 套接字的所有者、所有组和权限。**警告：** 两个服务器（Web 和 PHP）都必须具有该套接字的访问权限。

使用套接字时，你必须配置 `listen.allowed_clients` 以限制对 PHP 服务器的访问仅限于某些 IP 地址。

示例：`listen.allowed_clients = 127.0.0.1`

#### 静态或动态配置

你可以静态或动态地管理 PHP-FPM 的进程。

在静态模式中，`pm.max_children` 设置子进程的数量：

```bash
pm = static
pm.max_children = 10
```

此配置将启动 10 个进程。

在动态模式中，PHP-FPM 最多启动由 `pm.max_children` 值指定的进程数，从启动对应于 `pm.start_servers` 的一些进程开始，并至少保持 `pm.min_spare_servers` 个非活动进程和最多 `pm.max_spare_servers` 个非活动进程。

示例：

```bash
pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
```

PHP-FPM 将创建一个新进程来替换已处理相当于 `pm.max_requests` 数量的请求的进程。

默认情况下，`php-fpm` 将 `pm.max_requests` 设置为 0，意味着进程永远不会被回收。对于存在内存泄漏的应用程序，使用 `pm.max_requests` 选项可能会很有用。

还有第三种操作模式，即 `ondemand` (按需) 模式。此模式仅在收到请求时才启动一个进程。对于有大量流量的站点来说，这不是一种最佳模式，你应该将其保留用于特定需求（流量非常少的站点、管理后台等）。

!!! Note

    PHP-FPM 操作模式的配置对于确保 Web 服务器的最佳功能至关重要。

#### 进程状态

与 Apache 及其 `mod_status` 模块一样，PHP-FPM 提供了一个指示进程状态的页面。

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

#### 记录长请求日志

`slowlog` 指令指定接收过长请求日志记录的文件（例如，时间超过 `request_slowlog_timeout` 指令值的文件）。

生成文件的默认位置是 `/var/log/php-fpm/www-slow.log`。

```bash
request_slowlog_timeout = 5
slowlog = /var/log/php-fpm/www-slow.log
```

`request_slowlog_timeout` 的值为 0 将禁用日志记录。

### Nginx 集成

`nginx` 的默认设置已经包含使 PHP 与 PHP-FPM 配合工作所需的配置。

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

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

要使 `nginx` 处理 `.php` 文件，你必须将以下指令添加到站点配置文件中：

如果 PHP-FPM 在端口 9000 上监听：

```bash
location ~ \.php$ {
  include /etc/nginx/fastcgi_params;
  fastcgi_pass 127.0.0.1:9000;
}
```

如果 `php-fpm` 在 Unix 套接字上监听：

```bash
location ~ \.php$ {
  include /etc/nginx/fastcgi_params;
  fastcgi_pass unix:/run/php-fpm/www.sock;
}
```

### Apache 集成

配置 Apache 以使用 PHP 池非常简单。你必须使用代理模块配合 `ProxyPassMatch` 指令，例如：

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

### 稳固的 PHP 池配置

优化服务的请求数量，并分析 PHP 脚本使用的内存，以优化启动的最大线程数至关重要。

首先，你需要了解一个 PHP 进程的平均内存使用量，使用命令：

```bash
while true; do ps --no-headers -o "rss,cmd" -C php-fpm | grep "pool www" | awk '{ sum+=$1 } END { printf ("%d%s\n", sum/NR/1024,"Mb") }' >> avg_php_proc; sleep 60; done
```

运行一段时间后，这应该会给我们一个关于此服务器上 PHP 进程平均内存占用量的相当准确的概念。

本文档其余部分的结果是在满负载下每个进程 120 MB 的内存占用量。

在一台拥有 8 GB RAM 的服务器上，保留 1 GB 给系统，1 GB 给 OPCache（参见本文档的其余部分），剩下 6 GB 用于处理来自客户端的 PHP 请求。

你可以得出结论，此服务器最多可以接受 **50 个线程** `((6*1024) / 120)`。

一个针对此特定用例的良好的 `php-fpm` 配置将是：

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

`opcache` (Optimizer Plus Cache) 是我们可以影响的第一级缓存。

它将编译后的 PHP 脚本保存在内存中，这强烈影响了网页的执行（消除从磁盘读取脚本的时间 + 编译时间）。

要配置它，我们必须处理：

* 根据命中率 (hit ratio) 分配给 `opcache` 的内存大小，正确配置它
* 要缓存的 PHP 脚本数量（键的数量 + 最大脚本数量）
* 要缓存的字符串数量

要安装它：

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

* `opcache.memory_consumption` 对应于 `opcache` 所需的内存量（增加该值直到获得正确的命中率）。
* `opcache.interned_strings_buffer` 是要缓存的字符串数量。
* `opcache.max_accelerated_files` 接近于 `find ./ -iname "*.php"|wc -l` 命令的结果。

你可以参考一个包含 `phpinfo();` 的 `info.php` 页面来配置 `opcache`（例如参见 `Cached scripts` 和 `Cached strings` 的值）。

!!! Note

    在每次部署新代码时，需要清空 `opcache`（例如通过重启 `php-fpm` 进程）。

!!! Note

    不要低估通过正确设置和配置 `opcache` 可以实现的性能提升。
