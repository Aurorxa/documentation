---
title: Nginx
author: Ezequiel Bruni
contributors: Antoine Le Morvan, Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - nginx
  - web
---

# 如何在 Rocky Linux 上安装最新的 Nginx

## 简介

*Nginx* 是一款设计为快速、高效并与几乎所有技术兼容的 Web 服务器。我经常使用它，一旦你掌握了技巧——设置和配置它其实相当简单。为此，我编写了这份初学者指南。

以下是 Nginx 突出表现/具有的功能简要概述：

* 基础 Web 服务器
* 用于将流量定向到多个站点的反向代理
* 内置负载均衡器，用于管理到多个网站的流量
* 内置文件缓存以提升速度
* WebSockets
* FastCGI 支持
* 当然还有 IPv6

它很棒！所以只需要 `sudo dnf install nginx`，对吗？是的，基本上就是这样，但我们还包含了一些有用的提示来帮助你入门。

## 前提条件和假设

你需要：

* 一台联网的 Rocky Linux 机器或服务器。
* 对命令行有基本的了解。
* 能够以 root 身份运行命令，无论是作为 root 用户还是使用 `sudo`。
* 一个你选择的文本编辑器，无论是图形界面还是命令行的。在本教程中，我使用 `nano`。

## 安装和运行 Nginx

首先，确保你的机器已更新：

```bash
sudo dnf update
```

然后，安装 `nginx` 软件包：

```bash
sudo dnf install nginx
```

安装完成后，一次性启动 `nginx` 服务并启用它在重启时自动启动：

```bash
sudo systemctl enable --now nginx
```

要验证安装了最新版本的 *Nginx*（无论如何，来自 Rocky 仓库的最新版本），运行：

```bash
nginx -v
```

此后，你只需将 HTML 文件放入 `/usr/share/nginx/html/` 目录即可构建一个简单的静态网站。默认网站/虚拟主机的配置文件名为 "nginx.conf"，位于 `/etc/nginx/`。它还包含许多其他基本的 Nginx 服务器配置，因此即使你选择将实际的网站配置移到另一个文件中，你也应该保留 "nginx.conf" 的其余部分不变。

!!! Note

    本指南的旧版本描述了安装 nginx-mainline 的方法。这已不再是一个选项。在大多数情况下，Rocky 仓库中的 Nginx 版本完全足够，它提供了稳定的基础以及向后移植的安全补丁。那些仍想使用 nginx-mainline 的人可以通过搜索网络找到实现方法。然而，所有找到的操作文档都将适用于 Rocky Linux 8。请注意，使用 nginx-mainline 通常完全可行但不受支持。

## 配置防火墙

!!! Note

    如果你在容器（如 LXD/LXC 或 Docker）上安装 Nginx，你现在可以跳过这部分。防火墙应该由宿主机操作系统处理。

如果你尝试从另一台计算机查看你机器的 IP 地址或域名的网页，你很可能什么都看不到。好吧，如果你启用了防火墙就会出现这种情况。

要打开必要的端口以让你能真正"看到"你的网页，我们将使用 Rocky Linux 内置的防火墙，`firewalld`。用于执行此操作的 `firewalld` 命令是 `firewall-cmd`。有两种方法可以做到：官方方法和手动方法。*在这种情况下，官方方法是最好的，* 但为了将来参考，你应该了解两种方法。

官方方法打开防火墙允许 `http` 服务，这当然是处理网页的服务。只需运行：

```bash
sudo firewall-cmd --permanent --zone=public --add-service=http
```

让我们分解说明：

* `-–permanent` 标志告诉防火墙确保此配置在每次防火墙重启以及服务器重启时都使用。
* `–-zone=public` 告诉防火墙接受来自所有人的此端口的入站连接。
* 最后，`--add-service=http` 告诉 `firewalld` 将所有 HTTP 流量通过到服务器。

现在介绍手动方法。它基本相同，只是你要明确打开 HTTP 使用的端口 80。

```bash
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
```

* `–-add-port=80/tcp` 告诉防火墙接受通过端口 80 的入站连接，只要它们使用传输控制协议 (TCP)，这就是你在此情况下想要的。

对于 SSL/HTTPS 流量，只需再次运行命令并更改服务或端口号。

```bash
sudo firewall-cmd --permanent --zone=public --add-service=https
# 或者，在其他一些情况下：
sudo firewall-cmd --permanent --zone=public --add-port=443/tcp
```

这些配置在你强制要求之前不会生效。为此，告诉 `firewalld` 重新加载其配置，如下所示：

```bash
sudo firewall-cmd --reload
```

!!! Note

    现在，有很小的可能这不起作用。在那些罕见的情况下，使用古老的"关机再开机"方法让 `firewalld` 服从你的命令。

    ```bash
    systemctl restart firewalld
    ```

为确保端口已正确添加，运行 `firewall-cmd --list-all`。一个正确配置的防火墙看起来大致如下：

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

*现在* 你应该能看到一个看起来像这样的网页：

![Nginx 欢迎页面](nginx/images/welcome-nginx.png)

内容不多，但这意味着服务器正在运行。你也可以从命令行测试你的网页是否工作，使用：

```bash
curl -I http://[your-ip-address]
```

## 创建服务器用户并更改网站根目录

虽然你 *可以* 只需将你的网站放入默认目录然后就可以了（这对于运行在容器中或测试/开发服务器上的 *Nginx* 来说可能没问题），但这并不是我们所说的最佳实践。相反，一个好的做法是在你的系统上为你的网站创建一个特定的 Linux 用户，并将你的网站文件放在一个专门为该用户创建的目录中。

如果你想构建多个网站，你应该创建多个用户和根目录以实现组织和安全。

在本指南中，我将只有一个用户：一个名叫 "www" 的 handsome devil。决定把你的网站文件放在哪里会变得更加复杂。

你可以将你的网站文件放在几个地方，这取决于你的服务器设置。如果你在裸机（物理）服务器上，或者你直接在 VPS 上安装 `nginx`，你可能正在运行安全增强 Linux (SELinux)。SELinux 是一个做了很多事情来保护你机器的工具，但它也某种程度上规定了你可以将某些东西（如网页）放在哪里。

因此，如果你直接将 `nginx` 安装到你的机器上，你会想把网站放在默认根文件夹的子目录中。在这种情况下，默认根目录是 `/usr/share/nginx/html`，所以 "www" 用户的网站可能会放在 `/usr/share/nginx/html/www`。

然而，如果你在容器（如 LXD/LXC）中运行 `nginx`，SELinux 很可能 *不会* 安装，你可以将你的文件放在任何你喜欢的地方。在这种情况下，我喜欢将用户的所有网站文件放在正常 home 文件夹下的一个目录中，像这样：`/home/www/`。

不过，我将假设已安装 SELinux 继续本指南。只需根据你的用例更改你需要的内容。你也可以在[我们关于该主题的指南](../security/learning_selinux.md)中了解更多关于 SELinux 如何工作的信息。

### 创建用户

首先，创建我们将要使用的文件夹：

```bash
sudo mkdir /usr/share/nginx/html/www
```

接下来，创建 www 组：

```bash
sudo groupadd www
```

然后，我们创建用户：

```bash
sudo adduser -G nginx -g www -d /usr/share/nginx/html/www www --system --shell=/bin/false
```

该命令告诉机器：

* 创建一个名为 "www" 的用户（根据文本的中间部分），
* 将所有文件放在 `/usr/share/nginx/html/www`，
* 并将其添加到以下组："nginx" 作为补充组，"www" 作为主组。
* `--system` 标志表示该用户不是人类用户，而是预留给系统使用的。如果你想创建人类用户账户来管理不同的网站，那是另一整个指南的内容。
* `--shell=/bin/false` 确保没有人能 *尝试* 以 "www" 用户身份登录。

"nginx" 组做了一些真正的 magic。它允许 Web 服务器读取和修改属于 "www" 用户和 "www" 用户组的文件。有关更多信息，请参阅 Rocky Linux 的[用户管理指南](../../books/admin_guide/06-users.md)。

### 更改服务器根文件夹

现在你有了 fancy 的新用户账户，是时候让 `nginx` 在该文件夹中查找你的网站文件了。再次打开你最喜欢的文本编辑器。

目前，只需运行：

```bash
sudo nano /etc/nginx/conf.d/default.conf
```

文件打开后，找到类似于 `root   /usr/share/nginx/html;` 的行。将其更改为你选择的网站根文件夹，例如 `root   /usr/share/nginx/html/www;`（或者如果你像我一样在容器中运行 `nginx`，则为 `/home/www`）。保存并关闭文件，然后测试你的 `nginx` 配置，确保你没有漏掉分号或其他东西：

```bash
nginx -t
```

如果你收到以下成功消息，一切顺利：

```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

然后，使用以下命令对服务器进行软重启：

```bash
sudo systemctl reload nginx
```

!!! Note

    在不太可能的情况下软重启不起作用，使用以下命令给 `nginx` 一点 kick：

    ```bash
    sudo systemctl restart nginx
    ```

你的新根文件夹中的任何 HTML 文件现在都应该可以从……你的浏览器中浏览了。

### 更改文件权限

必须正确设置权限以确保 `nginx` 能够读取、写入和执行网站目录中的任何文件。

首先，确保根文件夹中的所有文件都属于服务器用户及其用户组：

```bash
sudo chown -R www:www /usr/share/nginx/html/www
```

然后，为确保想要浏览你网站的用户确实能看到页面，运行以下命令（是的，那些分号很重要）：

```bash
sudo find /usr/share/nginx/html/www -type d -exec chmod 555 "{}" \;
sudo find /usr/share/nginx/html/www -type f -exec chmod 444 "{}" \;
```

这基本上赋予每个人查看服务器上文件的权利，但不能修改它们。只有 root 和服务器用户才能做到这一点。

## 为你的站点获取 SSL 证书

截至目前，我们的[使用 certbot 获取 SSL 证书的指南](../security/generating_ssl_keys_lets_encrypt.md)已经更新，其中包含一些针对 `nginx` 的基本说明。去看看它，因为它包含了安装 certbot 以及生成证书的完整说明。

浏览器可能很快会阻止人们查看没有证书的网站，所以请确保为每个站点都获取一个证书。

## 其他配置选项和指南

* 如果你想了解如何让 *Nginx* 与 PHP 配合工作，特别是 PHP-FPM，请查看我们的 [Rocky Linux 上的 PHP 指南](../web/php.md)。
* 如果你想了解如何为多个网站设置 *Nginx*，我们现在有[一份关于该主题的指南](nginx-multisite.md)。

## SELinux 规则

请注意，当强制执行时，nginx proxy_pass 指令将失败并显示 "502 Bad Gateway" (502 错误的网关)。

你可以出于开发目的禁用 setenforce：

```bash
sudo setenforce 0
```

或者你可以在 `/var/log/audit/audit.log` 中为 nginx 相关的服务启用 `httpd` 或其他服务：

```bash
sudo setsebool httpd_can_network_connect 1 -P
```

## 结论

`nginx` 的基本安装和配置很简单，即使获取最新版本比应有的更复杂。但按照这些步骤操作，你将很快拥有一个运行中的最佳服务器选项之一。

现在你只需要去为自己构建一个网站？那大概再花十分钟？*Web 设计师默默哭泣*
