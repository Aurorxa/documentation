---
title: Apache 使用 'mod_ssl'
author: Garthus
contributors: Steven Spencer, David Hensley, Ganna Zhyrnova
update: 20-Jan-2022
---

# Rocky Linux 上 Apache Web 服务器环境中的 `mod_ssl`

Apache Web 服务器已经存在很多年了。`mod_ssl` 为 Web 服务器提供更高的安全性，并且几乎可以在任何版本的 Linux 上安装。

本步骤将帮助你在 Rocky Linux 和 Apache Web 服务器环境中运行 `mod_ssl`。

## 前提条件

* 一台工作站或服务器，最好已经安装了 Rocky Linux。
* 能够以 *root* 身份或使用 `sudo` 来提升权限。

## 安装 Rocky Linux 最小化版本

安装 Rocky Linux 时，我们使用了以下软件包组：

* Minimal (最小化)
* Standard (标准)

## 运行更新

首先，运行系统更新命令以使服务器重建仓库缓存，从而识别可用的软件包。

`dnf update`

## 启用仓库

对于传统的 Rocky Linux 服务器安装，所有必要的仓库都已就位。

## 检查可用的仓库

为确保无误，使用以下命令检查你的仓库列表：

`dnf repolist`

你将得到以下内容：

```bash
appstream                                                        Rocky Linux 8 - AppStream
baseos                                                           Rocky Linux 8 - BaseOS
extras                                                           Rocky Linux 8 - Extras
powertools                                                       Rocky Linux 8 - PowerTools
```

## 安装软件包

要安装 `mod_ssl`，运行：

`dnf install mod_ssl`

要启用 `mod_ssl` 模块，运行：

`apachectl restart httpd`
`apachectl -M | grep ssl`

你将看到：

  `ssl_module (shared)`

## 打开 TCP 端口 443

要允许 HTTPS 入站流量，运行：

```bash
firewall-cmd --zone=public --permanent --add-service=https
firewall-cmd --reload
```

添加此规则前，请确保你的目标是将网站对全世界开放！如果不是，请更改区域或配置防火墙以修正这一点。

此时，你应该能够通过 HTTPS 访问 Apache Web 服务器。输入 `https://your-server-ip` 或 `https://your-server-hostname` 来确认 `mod_ssl` 配置。

## 生成 SSL/TLS 证书

要为主机 rocky8 生成一个有效期为 365 天的自签名证书，运行：

`openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/httpd.key -x509 -days 365 -out /etc/pki/tls/certs/httpd.crt`

你将看到以下输出：

```bash
Generating a RSA private key
................+++++
..........+++++
writing new private key to '/etc/pki/tls/private/httpd.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:AU
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:LinuxConfig.org
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:rocky8
Email Address []:
```

此命令完成后，以下两个 SSL/TLS 文件将存在：

```bash
ls -l /etc/pki/tls/private/httpd.key /etc/pki/tls/certs/httpd.crt

-rw-r--r--. 1 root root 1269 Jan 29 16:05 /etc/pki/tls/certs/httpd.crt
-rw-------. 1 root root 1704 Jan 29 16:05 /etc/pki/tls/private/httpd.key
```

## 使用 SSL/TLS 证书配置 Apache Web 服务器

要将你新创建的 SSL/TLS 证书包含到 Apache Web 服务器配置中，通过运行以下命令打开 `ssl.conf` 文件：

`nano /etc/httpd/conf.d/ssl.conf`

更改以下行：

FROM (从):

```bash
SSLCertificateFile /etc/pki/tls/certs/localhost.crt
SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
```

TO (改为):

```bash
SSLCertificateFile /etc/pki/tls/certs/httpd.crt
SSLCertificateKeyFile /etc/pki/tls/private/httpd.key
```

通过运行以下命令重新加载 Apache Web 服务器：

`systemctl reload httpd`

## 测试 `mod_ssl` 配置

在 Web 浏览器中输入以下内容：

`https://your-server-ip` 或 `https://your-server-hostname`

## 将所有 HTTP 流量重定向到 HTTPS

通过运行以下命令创建一个新文件：

`nano /etc/httpd/conf.d/redirect_http.conf`

插入以下内容并保存文件，将 "your-server-hostname" 替换为你的主机名。

```bash
<VirtualHost _default_:80>

        Servername rocky8
        Redirect permanent / https://your-server-hostname/

</VirtualHost>
```

通过运行以下命令应用更改：

`systemctl reload httpd`

Apache Web 服务器现在会将任何来自 `http://your-server-hostname` 的流量重定向到 `https://your-server-hostname` URL。

## 最后步骤

你已经了解了如何安装和配置 `mod_ssl`，以及创建新的 SSL/TLS 证书以在 HTTPS 服务下运行 Web 服务器。

## 结论

本教程展示了 `mod_ssl` 的基本安装和使用。
