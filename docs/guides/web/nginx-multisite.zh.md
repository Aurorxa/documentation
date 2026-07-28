---
title: Nginx 多站点
author: Ezequiel Bruni
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - web
  - nginx
  - multisite
---

# 如何在 Rocky Linux 上为多个网站设置 Nginx

## 简介

这就是我承诺的 Rocky Linux 上 Nginx 多站点设置指南。我首先要给新手一点提示：其他人知道你们来这里是为了什么，所以直接往下滚吧。

嗨，新手们！Nginx *非常* 擅长的一件事是将流量从一个中心点定向到一台服务器或几台其他服务器上的多个网站和应用程序。这个功能被称为"反向代理"，Nginx 轻松做到这一点的能力是我开始使用它的原因之一。

在这里，我将向你展示如何在单个 Nginx 安装上管理多个网站，以及如何以一种简单、有组织的方式做到这一点，让你能够快速轻松地进行更改。

对于那些寻找 Apache 类似设置的人，请[参考本指南。](apache-sites-enabled.md)

我将解释 *很多* 细节……但最终，整个过程基本上涉及设置一些文件夹和制作一些小型文本文件。我们不会在本指南中使用过于复杂的网站配置，所以喝杯咖啡放松一下，找点乐趣。一旦你知道怎么做，每次都只需要几分钟。这个很简单。\*

\* 对于某些"简单"的定义。

## 前提条件和假设

这是你需要的一切：

- 一台连接到互联网的 Rocky Linux 服务器，上面已经运行了 Nginx。如果你还没有做到这一步，可以先按照[我们的 Nginx 安装指南](nginx-mainline.md)操作。
- 对在命令行中做事有一定的适应度，并且安装了像 `nano` 这样的终端文本编辑器。

    !!! tip "紧急情况下"

        ...你可以使用像 Filezilla 或 WinSCP 这样的工具——以及一个常规的基于 GUI 的文本编辑器——来复制大部分这些步骤，但在本教程中我们将用 nerdy 的方式做事。

- 至少一个指向你服务器用于测试网站的域名。你可以为另一个使用第二个域名或子域名。

    !!! tip

        如果你在本地服务器上做所有这些，根据需要调整你的 hosts 文件以创建模拟域名。说明见下文。

- 我们假设你在裸金属服务器或常规 VPS 上运行 Nginx，并且 SELinux 正在运行。所有说明默认都与 SELinux 兼容。
- *所有命令都必须以 root 身份运行，* 无论是通过以 root 用户身份登录，还是使用 `sudo`。

## 设置文件夹和测试站点

### 网站文件夹

首先，你需要几个用于网站文件的文件夹。当你首次安装 Nginx 时，所有"演示"网站文件都在 `/usr/share/nginx/html` 中。如果你只托管一个网站，这没问题，但我们要做得 fancy 一点。先忽略 `html` 目录，直接导航到它的父文件夹：

```bash
cd /usr/share/nginx
```

为本教程目的，测试域名将是 `site1.server.test` 和 `site2.server.test`，我们将相应地命名这些网站文件夹。当然，你应该将这些域名更改为你实际使用的任何域名。然而（这是我从业内聪明人那里学到的一个技巧），我们将把域名"反向"写出来。

例如，"yourwebsite.com" 将放在一个名为 `com.yourwebsite` 的文件夹中。请注意，你 *完全可以* 随意命名这些文件夹，但这种方法有一个很好的理由，我在下面概述了。

目前，只需创建你的文件夹：

```bash
mkdir -p test.server.site1/html
mkdir -p test.server.site2/html
```

所以该命令将创建例如 `test.server.site1` 文件夹，并在其中放入另一个名为 `html` 的文件夹。那就是你要放希望通过 Web 服务器提供的实际文件的地方。（你也可以称之为 "webroot" 或类似的名称。）

这样做的原因是你可以将不希望公开的网站相关文件放在父目录中，同时仍然将所有内容放在一个地方。

!!! Note

    `-p` 标志告诉 `mkdir` 命令创建你刚定义的路径中所有缺失的文件夹，这样你就不必逐个创建每个文件夹了。

对于此测试，我们保持"网站"本身非常简单。用你最喜欢的文本编辑器在第一个文件夹中创建一个 HTML 文件：

```bash
nano test.server.site1/html/index.html
```

然后粘贴以下 HTML 内容：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Site 1</title>
</head>
<body>
    <h1>This is Site 1</h1>
</body>
</html>
```

保存并关闭你的文件，然后用 `test.server.site2` 文件夹重复这些步骤，将上述 HTML 代码中的 "Site 1" 改为 "Site 2"。这只是为了我们稍后可以确保一切按预期工作。

你的测试网站已经完成，我们继续。

### 配置文件夹

现在让我们进入 Nginx 的设置和配置文件夹，这是我们将在本指南剩余部分工作的地方：

```bash
cd /etc/nginx/
```

如果你运行 `ls` 命令查看这里有哪些文件和文件夹，你会看到一堆不同的东西，今天大部分都不重要。需要注意的几个是：

- `nginx.conf` 是包含，你猜到了，默认 Nginx 配置的文件。我们稍后会编辑它。
- `conf.d` 是一个你可以放置自定义配置文件的目录。你 *可以* 将其用于网站，但最好用它来放置你希望在所有网站上都能用的功能特定设置。
- `default.d` 是一个如果你的服务器上只运行一个站点，或者你的服务器有一个"主"网站时，你的网站配置 *可能* 会去的地方。暂时不要去管它。

我们想创建两个新文件夹，名为 `sites-available` 和 `sites-enabled`：

```bash
mkdir sites-available
mkdir sites-enabled
```

我们要做的是将所有网站配置文件放在 `sites-available` 文件夹中。在那里，你可以根据需要长时间处理配置文件，直到准备好通过符号链接到 `sites-enabled` 文件夹来激活这些文件。

我将在下面展示它是如何工作的。目前，我们完成了创建文件夹。

!!! Note "为什么你（可能）想要反向写出你的域名："

    简单来说，这是一种组织方式，特别是在使用带有 Tab 补全功能的命令行时特别有用，但在基于 GUI 的应用程序中仍然相当有用。它是为在服务器上运行 *大量* 网站或应用程序的人设计的。

    基本上，你的所有网站文件夹（和配置文件）将按字母顺序组织；首先按顶级域名（例如 .com、.org 等），然后按主域名，然后按任何子域名。当你在搜索一长串域名时，以这种方式缩小搜索范围会更容易。

    这也使得通过命令行工具整理文件夹和配置文件更容易。要列出与特定域名关联的所有文件夹，你可能会运行：

    ```bash
    ls /usr/share/nginx/ | grep com.yoursite*
    ```

    这将输出类似的内容：

    ```
    com.yoursite.site1
    com.yoursite.site2
    com.yoursite.site3
    ```

## 设置配置文件

### 编辑 nginx.conf

默认情况下，Rocky Linux 的 Nginx 实现对所有 HTTP 流量开放，并将其全部定向到你可能在我们安装 Nginx 指南中看到的演示页面。我们不想那样。我们希望来自我们指定域名的流量去往我们指定的网站。

因此，从 `/etc/nginx/` 目录中，用你最喜欢的文本编辑器打开 `nginx.conf`：

```bash
nano nginx.conf
```

首先，找到看起来像这样的行：

```bash
include /etc/nginx/conf.d/*.conf;
```

并在这行 **正下方添加**：

```bash
include /etc/nginx/sites-enabled/*.conf;
```

这将使得我们的网站配置文件在准备好上线时被加载。

现在往下滚动到看起来像这样的部分，要么使用井号 ++#++ **将其注释掉**，要么删除它如果你愿意的话：

```bash
server {
    listen       80;
    listen       [::]:80;
    server_name  _;
    root         /usr/share/nginx/www/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    error_page 404 /404.html;
    location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    }
}
```

"注释掉"后的样子：

```bash
#server {
#    listen       80;
#    listen       [::]:80;
#    server_name  _;
#    root         /usr/share/nginx/www/html;
#
#    # Load configuration files for the default server block.
#    include /etc/nginx/default.d/*.conf;
#
#    error_page 404 /404.html;
#    location = /404.html {
#    }
#
#    error_page 500 502 503 504 /50x.html;
#    location = /50x.html {
#    }
#}
```

如果你是初学者，你可能想把注释的代码保留下来供参考，这也适用于文件更下方已经注释掉的示例 HTTPS 代码。

保存并关闭文件，然后使用以下命令重启服务器：

```bash
systemctl restart nginx
```

现在至少没有人能看到演示页面了。

### 添加网站配置文件

现在让我们让你的测试网站可以在服务器上访问。如前所述，我们将使用符号链接来完成，这样我们就有一个随时打开和关闭网站的简单方法。

!!! Note

    对于绝对的新手，符号链接基本上是一种让文件假装同时在两个文件夹中的方法。更改原始文件（或"目标"），它在所有链接到它的地方都会改变。如果你通过链接使用某个程序编辑文件，原始文件也会被改变。

    然而，如果你删除指向目标的链接，原始文件不会发生任何事情。这个技巧让我们能够将网站配置文件放在工作目录中 (`sites-available`)，然后通过从 `sites-enabled` 链接到这些文件来"激活"它们。

我来展示我的意思。为第一个网站创建一个配置文件：

```bash
nano sites-available/test.server.site1.conf
```

现在粘贴这段代码。这是你可能拥有的最简单的可工作的 Nginx 配置，对大多数静态 HTML 网站来说应该都很好：

```bash
server {
    listen 80;
    listen [::]:80;

    # virtual server name i.e. domain name #
    server_name site1.server.test;

    # document root #
    root        /usr/share/nginx/test.server.site1/html;

    # log files
    access_log  /var/log/nginx/www_access.log;
    error_log   /var/log/nginx/www_error.log;

    # Directives to send expires headers and turn off 404 error logging. #
    location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
        access_log off; log_not_found off; expires max;
    }
}
```

而且说实话，从 document root 往下的所有东西在技术上都是可选的。有用且推荐，但不是网站运行严格要求的。

不管怎样，保存并关闭文件，然后进入 `sites-enabled` 目录：

```bash
cd sites-enabled
```

现在，为你刚在 `sites-available` 文件夹中创建的配置文件创建一个符号链接：

```bash
ln -s ../sites-available/test.server.site1.conf
```

用 `nginx -t` 命令测试你的配置，如果收到一切正常的消息，重新加载服务器：

```bash
systemctl restart nginx
```

然后将你的浏览器指向你用于这个第一个站点的域名（我的情况是：site1.server.test），查找我们放在 HTML 文件中的 "This is Site 1" 消息。如果你的系统上安装了 `curl`，你可以运行 `curl site1.server.test` 并查看 HTML 代码是否加载到你的终端中。

!!! Note

    一些浏览器会（出于最好的意图）在你将服务器域名输入地址栏时强制你使用 HTTPS。如果你没有配置 HTTPS，那只会向你抛出错误。

    确保在你的浏览器地址栏中手动指定 "http://" 以避免此问题。如果那不起作用，清空缓存，或在此部分测试中使用一个不太挑剔的浏览器。我推荐 [Min](https://minbrowser.org)。

如果 *所有* 这些都顺利，*重复上述步骤，沿途更改文件名和配置文件的内容*。"site1" 到 "site2" 等等。一旦你为 Site 1 和 Site 2 都有了配置文件和符号链接，并重启了 Nginx，它应该看起来像这样：

![两个测试网站并排显示的截图](nginx/images/multisite-nginx.png)

!!! Note

    你也可以从 sites-enabled 目录外部使用 `ln -s` 命令的长格式来创建链接。它看起来像 `ln -s [source-file] [link]`。

    在此上下文中，那就是：

    ```bash
    ln -s /etc/nginx/sites-available/test.server.site1.conf /etc/nginx/sites-enabled/test.server.site1.conf
    ```

### 禁用网站

如果你需要在再次上线之前停止你的某个网站以进行处理，只需删除 sites-enabled 中的符号链接：

```bash
rm /etc/nginx/sites-enabled/test.server.site1.conf
```

然后像往常一样重启 Nginx。要让网站重新上线，你需要重新创建符号链接，并再次重启 Nginx。

## 可选：编辑你的 Hosts 文件

这部分绝对是给初学者的。其他人都可以跳过。

所以这一节 *仅* 适用于你在本地开发环境中尝试本指南的情况。也就是说，如果你在你的工作站上，或者在你本地家庭或商业网络中的另一台机器上运行你的测试服务器。

由于将外部域名指向你的本地机器很麻烦（而且如果你不知道自己在做什么，可能会有危险），你可以设置一些可以在你的本地网络上有用但在其他地方无效的"假"域名。

最简单的方法是使用你计算机上的 hosts 文件。hosts 文件实际上只是一个可以覆盖 DNS 设置的文本文件。也就是说，你可以手动指定一个域名对应任何你想要的 IP 地址。不过，它 *只会* 在那台计算机上起作用。

所以在 Mac 和 Linux 上，hosts 文件在 `/etc/` 目录中，可以通过命令行非常容易地编辑（你需要 root 访问权限）。假设你在 Rocky Linux 工作站上工作，只需运行：

```bash
nano /etc/hosts
```

在 Windows 上，hosts 文件位于 `C:\Windows\system32\drivers\etc\hosts`，只要你具有管理员访问权限，就可以使用任何你想要的 GUI 文本编辑器。

所以如果你在 Rocky Linux 计算机上工作，并在同一台机器上运行你的 Nginx 服务器，你只需打开文件，定义你想要的域名/IP 地址。如果你在同一台机器上运行你的工作站和测试服务器，那就是：

```bash
127.0.0.1           site1.server.test
127.0.0.1           site2.server.test
```

如果你在网络上的另一台机器上运行你的 Nginx 服务器，只需使用那台机器的地址，例如：

```bash
192.168.0.45           site1.server.test
192.168.0.45           site2.server.test
```

然后你就可以将你的浏览器指向这些域名，它应该按预期工作。

## 为你的站点设置 SSL 证书

去查看[我们使用 Let's Encrypt 和 certbot 获取 SSL 证书的指南](../security/generating_ssl_keys_lets_encrypt.md)。那里的说明可以很好地工作。

## 结论

记住，这里大多数文件夹/文件组织和命名约定在技术上都是可选的。你的网站配置文件主要只需要放在 `/etc/nginx/` 中的任何位置，并且 `nginx.conf` 需要知道这些文件在哪里。

实际的网站文件应该放在 `/usr/share/nginx/` 中的某个地方，其余都是锦上添花。

试试看，做一些 Science^TM^，别忘了在重启 Nginx 之前运行 `nginx -t` 以确保你没有漏掉分号或其他东西。这会节省你大量时间。
