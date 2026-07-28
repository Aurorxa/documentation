---
title: 生成 SSL 密钥 - Let's Encrypt
author: Steven Spencer
contributors: wsoyinka, Antoine Le Morvan, Ezequiel Bruni, Andrew Thiesen, Ganna Zhyrnova
tested*with: 8.5
tags:
  - security
  - ssl
  - certbot
---

## 前提条件与假设

- 熟悉命令行
- 熟悉使用 SSL 证书保护网站是加分项
- 了解命令行文本编辑器（本示例使用 *vi*）
- 一个对世界开放并在端口 80（`http`）上运行的 Web 服务器
- 熟悉 *SSH*（安全外壳）并能够使用 *SSH* 访问您的服务器
- 所有命令都假设您是 root 用户或使用了 `sudo` 获得 root 访问权限

## 简介

保护网站最流行的方式之一是使用 Let's Encrypt SSL 证书，这也是免费的。

这些是真正的证书，不是自签名或无效的，因此对于低预算的安全解决方案来说非常棒。本文档将引导您完成在 Rocky Linux Web 服务器上安装和使用 Let's Encrypt 证书的过程。

## 安装

要执行下一步操作，使用 `ssh` 登录到您的服务器。如果您的服务器完全限定的 DNS 名称是 <www.myhost.com>，那么您将使用：

```bash
ssh -l root www.myhost.com
```

或者，如果您必须先以非特权用户身份访问您的服务器。使用您的用户名：

```bash
ssh -l username www.myhost.com
```

然后：

```bash
sudo -s
```

在这种情况下，您将需要您的用户凭据以 root 身份访问系统。

Let's Encrypt 使用一个名为 *certbot* 的软件包，您通过 EPEL 仓库安装。首先添加这些：

```bash
dnf install epel-release
```

安装适当的软件包，取决于您使用 Apache 还是 Nginx 作为 Web 服务器。对于 Apache：

```bash
dnf install certbot python3-certbot-apache
```

对于 Nginx，只需更换一个软件包：

```bash
dnf install certbot python3-certbot-nginx
```

如果需要，您随时可以安装两个服务器模块。

!!! Note

    本指南的早期版本需要 *certbot* 的 snap 软件包版本，这在当时是必需的。RPM 版本最近已经重新测试，现在可以正常工作了。话虽如此，Certbot 强烈建议使用 [snap 安装过程](https://certbot.eff.org/instructions?ws=apache&os=centosrhel8)。Rocky Linux 8 和 9 在 EPEL 中提供了 *certbot*，因此我们在此展示了该过程。如果您想使用 Certbot 推荐的过程，只需按照该过程操作即可。

## 为 Apache 服务器获取 Let's Encrypt 证书

您可以通过两种方式获取 Let's Encrypt 证书：使用命令为您更改 `http` 配置文件，或者仅获取证书。如果您正在使用 [Apache Web Server Multi-Site Setup](../web/apache-sites-enabled.md) 过程中为一个或多个站点建议的多站点设置过程，那么只需获取证书。

此处假设是多站点设置，因此接下来的说明将仅获取证书。如果您运行的是默认配置的独立 Web 服务器，您可以一步完成获取证书和更改配置文件：

```bash
certbot --apache
```

这确实是完成事情的最简单方法。但是，有时您希望采取更手动的方式并获取证书。要仅获取证书，请使用以下命令：

```bash
certbot certonly --apache
```

这些命令将生成一组您需要回答的提示。第一个是提供用于重要信息的电子邮件地址：

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator apache, Installer apache
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): yourusername@youremaildomain.com
```

下一个要求您阅读并接受订阅者协议的条款。阅读协议后，回答 'Y' 以继续：

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

下一个是请求与电子前哨基金会共享您的电子邮件。根据您的偏好回答 'Y' 或 'N'：

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

下一个提示要求您识别您想要证书的域。它可能会根据您运行的 Web 服务器在列表中显示一个域。如果是这样，输入与您正在获取证书的域对应的数字。在这种情况下，只有一个选项 ('1') 存在：

```bash
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: your-server-hostname
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):
```

如果一切顺利，您将收到以下消息：

```bash
Requesting a certificate for your-server-hostname
Performing the following challenges:
http-01 challenge for your-server-hostname
Waiting for verification...
Cleaning up challenges
Subscribe to the EFF mailing list (email: yourusername@youremaildomain.com).

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your-server-hostname/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your-server-hostname/privkey.pem
   Your certificate will expire on 2021-07-01. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

## 站点配置 - `https`

将配置文件应用到我们的站点几乎与使用从其他提供商购买的 SSL 证书的过程相同。

单个 PEM（Privacy Enhanced Mail，隐私增强邮件）文件包含证书和链文件。这是现在所有证书文件的通用格式。尽管它引用了 "Mail"，但它只是一种证书文件类型。下面是配置文件的说明，以及正在发生的事情的描述：

!!! info

    之前，此文档在配置中包含一行用于 `SSLCertificateChainFile` 指令。自 Apache 2.4.8 版本起，该指令已被弃用，因为 `SSLCertificateFile` 指令[现在扩展到包括中间 CA 文件](https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatechainfile)。

```bash
<VirtualHost *:80>
        ServerName your-server-hostname
        ServerAdmin username@rockylinux.org
        Redirect / https://your-server-hostname/
</VirtualHost>
<VirtualHost *:443>
        ServerName your-server-hostname
        ServerAdmin username@rockylinux.org
        DocumentRoot /var/www/sub-domains/com.yourdomain.www/html
        DirectoryIndex index.php index.htm index.html
        Alias /icons/ /var/www/icons/
        # ScriptAlias /cgi-bin/ /var/www/sub-domains/com.yourdomain.www/cgi-bin/

        CustomLog "/var/log/httpd/com.yourdomain.www-access_log" combined
        ErrorLog  "/var/log/httpd/com.yourdomain.www-error_log"

        SSLEngine on
        SSLProtocol all -SSLv2 -SSLv3 -TLSv1
        SSLHonorCipherOrder on
        SSLCipherSuite EECDH+ECDSA+AESGCM:EECDH+aRSA+AESGCM:EECDH+ECDSA+SHA384:EECDH+ECDSA+SHA256:EECDH+aRSA+SHA384:EECDH+aRSA+SHA256:EECDH+aRSA+RC4:EECDH:EDH+aRSA:RC4:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!SRP:!DSS

        SSLCertificateFile /etc/letsencrypt/live/your-server-hostname/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/your-server-hostname/privkey.pem

        <Directory /var/www/sub-domains/com.yourdomain.www/html>
                Options -ExecCGI -Indexes
                AllowOverride None

                Order deny,allow
                Deny from all
                Allow from all

                Satisfy all
        </Directory>
</VirtualHost>
```

以下是正在发生的事情：

- 即使端口 80（标准 `http`）正在监听，您正在将所有流量重定向到端口 443（`https`）
- SSLEngine on - 表示使用 SSL
- SSLProtocol all -SSLv2 -SSLv3 -TLSv1 - 表示使用可用的协议，除了那些被发现存在漏洞的。您需要定期研究当前可接受使用的协议。
- SSLHonorCipherOrder on - 这涉及下一行关于密码套件的问题，并表示按列出的顺序处理它们。这是您需要定期审查您想要包含的密码套件的另一个领域
- SSLCertificateFile - 这是 PEM 文件，包括站点证书**和**中间证书。
- SSLCertificateKeyFile - 用于私钥的 PEM 文件，在 *certbot* 请求期间生成。

当您完成所有更改后，重启 *httpd*，如果它启动，测试您的站点以确保您现在显示有效的证书文件。如果是这样，您就可以继续下一步：自动化。

## 将 *certbot* 与 Nginx 一起使用

简短说明：将 *certbot* 与 Nginx 一起使用与 Apache 基本相同。以下是该指南的简短版本：

运行此命令以开始：

```bash
certbot --nginx
```

您需要输入您的电子邮件地址和您想要证书的站点。假设您至少配置了一个站点（域名指向服务器），您将看到一个列表：

```bash
1. yourwebsite.com
2. subdomain.yourwebsite.com
```

如果您有多个站点，按与您想要证书的站点对应的数字。

其余文本类似于上面的内容。结果会有点不同。如果您有一个看起来像这样的 Nginx 配置文件：

```bash
server {
    server_name yourwebsite.com;

    listen 80;
    listen [::]:80;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}

```

在 *certbot* 处理之后，它将看起来像这样：

```bash
server {
    server*name  yourwebsite.com;

    listen 443 ssl; # managed by Certbot
    listen [::]:443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/yourwebsite.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/yourwebsite.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}

server {
    if ($host = yourwebsite.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  listen 80;
  listen [::]:80;
  server_name yourwebsite.com;
    return 404; # managed by Certbot
}
```

如果您将 Nginx 用作反向代理，您可能需要更改新的配置文件来修复一些 *certbot* 自身无法完美处理的事项。

## 自动化 Let's Encrypt 证书续期

!!! note

    在这些示例中，将出现的 "your-server-hostname" 替换为实际的域名或主机名。

安装 *certbot* 的好处是 Let's Encrypt 证书将自动续期。您无需创建一个过程来执行此操作。您*确实*需要测试续期：

```bash
certbot renew --dry-run
```

当您运行此命令时，您将获得一个显示续期过程的良好输出：

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/your-server-hostname.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator apache, Installer apache
Account registered.
Simulating renewal of an existing certificate for your-server-hostname
Performing the following challenges:
http-01 challenge for your-server-hostname
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of apache server; fullchain is
/etc/letsencrypt/live/your-server-hostname/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/your-server-hostname/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

您可以通过以下方式之一续期 *certbot* 证书：

- 通过列出 `/etc/crontab/` 的内容
- 通过列出 `/etc/cron.*/*` 的内容
- 通过运行 `systemctl list-timers`

使用 `systemctl list-timers` 方法，您可以看到 *certbot* 存在，并且其安装使用了 `snap` 过程：

```bash
sudo systemctl list-timers
Sat 2021-04-03 07:12:00 UTC  14h left   n/a                          n/a          snap.certbot.renew.timer     snap.certbot.renew.service
```

## 结论

Let's Encrypt SSL 证书是使用 SSL 保护您的网站的另一种选择。安装后，系统提供证书的自动续期，并将加密到您网站的流量。

Let's Encrypt 证书适用于标准的 DV（Domain Validation，域名验证）证书。无法将它们用于 OV（Organization Validation，组织验证）或 EV（Extended Validation，扩展验证）证书。
