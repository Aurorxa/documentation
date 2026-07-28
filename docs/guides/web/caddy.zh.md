---
title: Caddy Web 服务器
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.3, 10.0
tags:
  - web
---

## 简介

*Caddy* 是一款为现代 Web 应用程序设计的 Web 服务器。Caddy 配置简单，并具有自动 Let's Encrypt 支持，因此你的网站始终默认安全。它是作者的首选 Web 服务器。

以下是 Caddy 功能的简要概述：

* 基础 Web 服务器
* 用于将流量定向到多个站点的反向代理
* 支持多种工作负载的模块，包括 TCP、SSH 等
* 内置负载均衡器，用于管理到多个网站的流量
* 内置的自动化 Let's Encrypt 支持
* 通过 API 以编程方式重新配置服务器
* PHP FastCGI 支持
* 以及 IPv6

## 前提条件和假设

你需要：

* 一台联网的 Rocky Linux 机器或服务器。
* 对命令行有基本的了解。
* 能够以 root 用户身份或使用 `sudo` 运行命令。
* 一个你选择的文本编辑器，无论是图形界面还是命令行的。在本教程中，作者使用 `vim`。
* 一个指向你服务器公网 IP 地址的域名或其他主机名。

## 安装 Caddy

首先，确保你的机器有最新的更新：

```bash
sudo dnf update
```

然后，安装 `epel-release` 软件仓库：

```bash
sudo dnf install -y epel-release
```

如果你运行的是 Rocky Linux 10，启用 Copr 仓库：

```bash
sudo dnf copr enable @caddy/caddy
```

接下来，安装 `caddy` Web 服务器：

```bash
sudo dnf install -y caddy
```

## 配置防火墙

如果你尝试从另一台计算机查看你机器的 IP 地址或域名的网页，你很可能什么都看不到。如果你启用了防火墙，就会出现这种情况。

要打开必要的端口以真正"看到"你的网页，你将使用 Rocky Linux 内置的防火墙，`firewalld`。用于执行此操作的 `firewalld` 命令是 `firewall-cmd`。

要打开 `http` 和 `https` 服务（处理网页的服务），运行：

```bash
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
```

分解说明：

* `-–permanent` 标志告诉防火墙在每次防火墙重启以及服务器重启时应用此配置。
* `–-zone=public` 告诉防火墙允许来自所有人的此端口的入站连接。
* 最后，`--add-service=http` 和 `--add-service=https` 告诉 `firewalld` 将所有 HTTP 和 HTTPS 流量传递给服务器。

这些配置在你强制要求之前不会生效。为此，告诉 `firewalld` 重新加载其配置：

```bash
sudo firewall-cmd --reload
```

!!! Note

    现在，有很小的可能这不起作用。在那些罕见的情况下，使用古老的"关机再开机"方法让 `firewalld` 服从你的命令。

    ```bash
    systemctl restart firewalld
    ```

为确保端口已被允许，运行 `firewall-cmd --list-all`。一个正确配置的防火墙看起来大致如下：

```bash
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp9s0
  sources:
  services: cockpit dhcpv6-client ssh http https
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

防火墙方面应该就这些了。

## 配置 Caddy

与传统的 Web 服务器（如 Apache 和 Nginx）不同，Caddy 的配置格式要简单得多。你不再需要配置细节，例如 Web 服务器的线程模型或 SSL 证书，除非你想那么做。

要编辑 Caddy 配置文件：

```bash
sudo vim /etc/caddy/Caddyfile
```

一个最小的静态 Web 服务器配置可以类似这样：

```bash
example.com {
    root * /usr/share/caddy/example.com
    file_server
}
```

将 "example.com" 替换为指向你的服务器的主机名。

你还必须将一个网站添加到 Caddy "root" 目录中的文件夹。为简单起见，添加一个单页静态网站：

```bash
mkdir -p /usr/share/caddy/example.com
echo '<h1>Hi!</h1>' | sudo tee /usr/share/caddy/example.com/index.html
```

之后，启用 Caddy 的 systemd 服务：

```bash
sudo systemctl enable --now caddy
```

在一分钟内，Caddy 将从 Let's Encrypt 获取 SSL 证书。然后，你可以在浏览器中查看你刚设置的网站：

![Caddy 提供我们的演示网站](../images/caddy_example.png)

它应该有一个在每种现代浏览器中都能正常工作的 SSL 锁，不仅如此，在 [Qualys SSL 服务器测试](https://www.ssllabs.com/ssltest/)中还能获得 A+ 评级。

## 可选：PHP FastCGI

如前所述，Caddy 支持用于 PHP 的 FastCGI 支持。好消息是，与 Apache 和 Nginx 不同，Caddy 自动处理 PHP 文件扩展名。

要安装 PHP，首先添加 Remi 仓库（注意：如果你运行的是 Rocky Linux 8.x 或 9.x，在下面的 "release-" 后面替换为 8 或 9）：

```bash
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-10.rpm
```

接下来，我们需要安装 PHP（注意：如果你使用其他版本的 PHP，将 php85 替换为你想要的版本）：

```bash
sudo dnf install -y php85-php-fpm
```

如果你需要额外的 PHP 模块（例如 GD），将它们添加到上述命令中。

然后，我们需要配置 PHP 以在 TCP 套接字上监听：

```bash
sudo vim /etc/opt/remi/php85/php-fpm.d/www.conf
```

接下来，找到这一行：

```bash
listen = /var/opt/remi/php85/run/php-fpm/www.sock
```

将其替换为：

```bash
listen = 127.0.0.1:9000
```

现在你可以启用并启动 `php-fpm`：

```bash
sudo systemctl enable --now php85-php-fpm
```

然后保存并退出 `www.conf` 文件，打开 Caddyfile：

```bash
sudo vim /etc/caddy/Caddyfile
```

导航到我们之前创建的 server block：

```bash
example.com {
    root * /usr/share/caddy/example.com
    file_server
}
```

在 "file_server" 行之后添加以下行：

```bash
    php_fastcgi 127.0.0.1:9000
```

你启用 PHP 的 server block 将看起来像这样：

```bash
example.com {
    root * /usr/share/caddy/example.com
    file_server
    php_fastcgi 127.0.0.1:9000
}
```

然后保存并退出 Caddyfile，并重启 Caddy：

```bash
sudo systemctl restart caddy
```

要测试 PHP 是否工作，让我们添加一个简单的 PHP 文件：

```bash
echo "<?php phpinfo(); ?>" >> /usr/share/caddy/rockyexample.duckdns.org/phpinfo.php
```

在浏览器中打开你创建的文件，你应该会看到 PHP 信息页面：

![Caddy 提供我们的 PHP 文件](../images/caddy_php.png)

## 结论

Caddy 的基本安装和配置非常简单。那些你花数小时配置 Apache 的日子已经一去不复返了。是的，Nginx 无疑是一种改进，但它仍然缺乏 Caddy 内置的现代必要功能，如 Let's Encrypt 和 Kubernetes ingress 支持，而在 Nginx（和 Apache）上你必须单独添加这些功能。

作者自 2019 年以来一直使用 Caddy 作为首选 Web 服务器，它实在是太棒了。事实上，每当我与 Apache、Nginx 或 IIS 打交道时，几乎是像坐时光机回到 2010 年或更早的时代。
