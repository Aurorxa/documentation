---
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
title: 第 2.2 部分 Web 服务器 Nginx 
---

## Nginx Web 服务器

在本章中，您将学习关于 Nginx Web 服务器的知识。

****

**目标**：您将学习如何：

:heavy_check_mark: 安装和配置 Nginx

:checkered_flag: **nginx**, **http**

**知识**：:star: :star:  
**复杂性**：:star: :star:  

**阅读时间**：15 分钟

****

### 概述

**Nginx** 是一个基于 **BSD 许可证**的**自由 HTTP Web 服务器**。它最初由 Igor Sysoev 于 2002 年在俄罗斯开发。除了 Web 服务器的标准功能外，Nginx 还为 **HTTP** 协议提供了**反向代理**，以及为 **POP** 和 **IMAP** 消息协议提供了代理。

Nginx 服务器的发展是对 **C10K** 问题的回应，即支持一万个并发连接（现代 Web 的标准）。这对 Web 服务器来说是一个真正的挑战。

商业支持可从 Nginx Inc 获得。

与 Apache Web 服务器相比，该服务器的内部架构实现了**非常高的性能**和**低内存消耗**。

补充 Nginx 内核基本功能的模块在编译时绑定。这意味着无法动态激活或停用。

一个 master 进程控制服务器进程，使得可以在**不停服务的情况下修改配置或更新软件**。

Nginx 在最繁忙网站上拥有 28% 的重要市场份额，仅次于 Apache（41%）。

#### 特性

Nginx 提供以下基本功能：

* 静态 Web 页面托管
* 自动索引页面生成
* 带缓存的加速反向代理
* 负载均衡
* 容错
* 对 FastCGI、uWSGI、SCGI 和 Memcached 缓存服务器的缓存支持
* 各种过滤器：gzip、xslt、ssi、图像转换等
* SSL/TLS 和 SNI 支持
* HTTP/2 支持

其他特性：

* 按名称或 IP 地址托管
* 客户端连接的 Keepalive 管理
* 日志管理：syslog、轮转、缓冲
* URI 重写
* 访问控制：按 IP、密码等
* FLV 和 MP4 流媒体

### 安装

Nginx 可直接从 app stream 仓库获得，较新版本可作为 dnf module 使用。

```bash
sudo dnf install nginx
sudo systemctl enable nginx --now
```

### 配置

Nginx 配置位于 `/etc/nginx/nginx.conf`。

此配置文件是一个全局服务器配置文件。设置影响整个服务器。

!!! NOTE

    Apache 管理员熟知的 .htaccess 文件功能在 Nginx 中不存在！

以下提供的是去掉所有注释的 `nginx.conf` 文件：

```bash
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;
        error_page 404 /404.html;
        location = /404.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
```

默认配置指南：

| 指令                       | 描述 |
|-----------------------------|-------------|
| `user`                      | 定义进程所有者 `user` 和 `group`。如果未指定 group，则使用与用户同名的 group。 |
| `worker_processes`          | 定义进程数量。最佳值取决于许多因素，例如 CPU 核心数量和硬盘规格。如有疑问，Nginx 文档建议起始值等同于可用的 CPU 核心数（auto 值将尝试确定此值）。 |
| `pid`                       | 定义一个用于存储 PID 值的文件。 |
| `worker_connections`        | 设置一个 worker 进程可以打开的最大同时连接数（到客户端和被代理服务器）。 |
| `tcp_nopush`                | `tcp_nopush` 与 sendfile 选项不可分割。用于优化同时发送的信息量。数据包仅在达到其最大大小时才发送。 |
| `tcp_nodelay`               | 激活 `tcp_nodelay` 会强制立即发送 socket 中的数据，无论数据包大小如何，这与 `tcp_nopush` 的作用相反。 |
| `sendfile`                  | 优化静态文件的发送（此选项对于反向代理配置不是必需的）。如果启用了 sendfile，Nginx 确保所有数据包在发送到客户端之前已完成（得益于 `tcp_nopush`）。当最后一个数据包到达时，Nginx 禁用 `tcp_nopush` 并使用 `tcp_nodelay` 强制发送数据。 |
| `keepalive_timeout`         | 关闭空闲连接前的最大时间。                                                |
| `types_hash_max_size`       | Nginx 维护包含静态信息的哈希表。设置哈希表的最大大小。 |
| `include`                   | 在配置中包含另一个或多个匹配所提供模板的文件。               |
| `default_type`              | 请求的默认 MIME 类型。                                                                    |
| `ssl_protocols`             | 接受的 TLS 协议版本。                                                                    |
| `ssl_prefer_server_ciphers` | 优先使用服务器密码套件而非客户端密码套件。                                                 |
| `access_log`                | 配置访问日志（参见"日志管理"段落）。                                            |
| `error_log`                 | 配置错误日志（参见"日志管理"段落）。                                             |
| `gzip`                      | ngx_http_gzip_module 是一个过滤器，以 gzip 格式压缩传输的数据。              |
| `gzip_disable`              | 基于正则表达式禁用 gzip。                                                        |

Nginx 配置的结构是：

```text
# 全局指令

events {
    # worker 配置
}

http {
    # http 服务配置

    # 配置第一个监听 80 端口的服务器
    server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;
        index index.html index.htm;
        server_name _;
        location / {
            try_files $uri $uri/ =404;
        }
    }
}

mail {
    # 邮件服务配置

# 全局邮件服务指令
   server {
        # 第一个监听 pop 协议的服务器
        listen     localhost:110;
        protocol   pop3;
        proxy      on;
   }


   server {
        # 第二个监听 imap 协议的服务器
       listen     localhost:143;
       protocol   imap;
       proxy      on;
   }
}
```

### HTTPS 配置

要配置 HTTPS 服务，您必须添加一个 server 块或修改现有块。一个 server 块可以同时监听 443 端口和 80 端口。

例如，您可以将此块添加到新的 `/etc/nginx/conf.d/default_https.conf` 文件：

```bash
server {
    listen              443 ssl default_server;
    ssl_protocols       TLSv1.3 TLSv1.2 TLSv1.1
    ssl_certificate     /path/to/cert.pem;
    ssl_certificate_key /path/to/key.key;
    root                /var/www/html;
    index               index.html index.htm;
    server_name         _;
    location / {
        try_files       $uri $uri/ =404;
    }
}
```

或者您可以修改默认 server 以支持 HTTPS：

```bash
server {
    listen              80;
    listen              443 ssl;
    server_name         _;
    ssl_protocols       TLSv1.3 TLSv1.2 TLSv1.1
    ssl_certificate     /path/to/cert.pem;
    ssl_certificate_key /path/to/key.key;
    ...
}
```

### 日志管理

您可以为错误日志配置 `error_log` 指令。

`error_log` 指令的语法：

```bash
error_log file [level];
```

第一个参数定义一个接收错误日志的文件。

第二个参数确定日志级别：debug、info、notice、warn、error、crit、alert 或 emerg（参见我们管理员指南的 syslog 章节）。

使用 "syslog:" 前缀可将日志发送到 syslog。

```bash
access_log syslog:server=192.168.1.100:5514,tag=nginx debug;
```

### Nginx 作为反向代理

反向代理功能通过 `ngx_http_upstream_module` 实现。它允许您定义服务器组，然后由 `proxy_pass` 或 `fastcgi_pass` 指令、`memcached_pass` 等调用。

基本配置示例，将 2/3 的负载分发给第一个服务器，1/3 分发给第二个应用服务器：

```bash
    upstream frontservers {
        server front1.rockylinux.lan:8080       weight=2;
        server front2.rockylinux.lan:8080       weight=1;
    }

    server {
        location / {
            proxy_pass http://docs.rockylinux.lan;
        }
    }
```

您可以将服务器声明为备份：

```bash
    upstream frontservers {
        ...
        server front3.rockylinux.lan:8080   backup;
        server front4.rockylinux.lan:8080   backup;
    }
```

server 指令接受许多参数：

* `max_fails=attempt次数`: 设置在 `fail_timeout` 参数定义的时间段内必须失败的连接尝试次数，超过后服务器被视为不可用。默认值为 1；0 禁用此功能。
* `fail_timeout=时间`: 设置一个时间段，在该时间段内，指定数量的连接会导致服务器不可用，并设置服务器被视为不可用的时间长度。默认值为 10 秒。
