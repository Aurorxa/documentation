---
title: DokuWiki 服务器
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - wiki
  - documentation
---

## 前置条件和假设

- 在服务器、容器或虚拟机上安装了 Rocky Linux 实例
- 能舒适地使用编辑器从命令行修改配置文件（此处的示例使用 `vi`，但您可以替换为您喜欢的编辑器）
- 了解一些关于 Web 应用程序和设置的知识
- 使用 [Apache Sites Enabled](../web/apache-sites-enabled.md) 进行设置。如有必要，请查看该文档。
- 本文档将全程使用 "example.com" 作为域名
- 您必须是 root 或能够使用 `sudo` 提升权限
- 假设操作系统是全新安装的，但这不是必需的

## 简介

文档在组织中可以以多种形式存在。拥有一个可以引用的文档仓库是非常宝贵的。Wiki（在夏威夷语中意为_快_）是一种将文档、流程说明、企业知识库，甚至代码示例集中保存的方式。即使保密地维护 Wiki 的 IT 专业人员，也能拥有一个防遗忘的内置保险。

DokuWiki 是一个成熟、快速的 Wiki，运行无需数据库，具有内置的安全功能，且部署并不复杂。更多信息，请查看其[网页](https://www.dokuwiki.org/dokuwiki)。

DokuWiki 是众多可用 Wiki 中的一个，而且是一个优秀的选择。一个重大的优点是 DokuWiki 相对轻量，可以在已运行其他服务的服务器上运行，只要有足够的空间和内存。

## 安装依赖项

DokuWiki 的最低 PHP 版本现在是 8。Rocky Linux 10 默认使用 PHP 8.3。请注意，此处列出的一些软件包可能已经存在：

```bash
dnf install tar wget httpd php php-gd php-xml php-json php-mbstring
```

接受并安装这些软件包列出的任何额外依赖项。

## 创建目录并更改配置

### Apache 配置

如果您已经阅读了 [Apache Sites Enabled](../web/apache-sites-enabled.md) 过程，您就知道需要创建一些目录。从 `httpd` 配置目录的添加开始：

```bash
mkdir -p /etc/httpd/{sites-available,sites-enabled}
```

您需要编辑 `httpd.conf` 文件：

```bash
vi /etc/httpd/conf/httpd.conf
```

将以下内容添加到文件的最后一行：

```bash
Include /etc/httpd/sites-enabled
```

在 sites-available 中创建站点配置文件：

```bash
vi /etc/httpd/sites-available/com.example
```

该配置文件将类似于以下内容：

```apache
<VirtualHost *>
  ServerName    example.com
  DocumentRoot  /var/www/sub-domains/com.example/html

  <Directory ~ "/var/www/sub-domains/com.example/html/(bin/|conf/|data/|inc/)">
      <IfModule mod_authz_core.c>
                AllowOverride All
          Require all denied
      </IfModule>
      <IfModule !mod_authz_core.c>
          Order allow,deny
          Deny from all
      </IfModule>
  </Directory>

  ErrorLog   /var/log/httpd/example.com_error.log
  CustomLog  /var/log/httpd/example.com_access.log combined
</VirtualHost>
```

请注意，此处的 "AllowOverride All" 允许 `.htaccess`（目录特定安全）文件工作。

现在将配置文件链接到 sites-enabled，但暂且不要启动 Web 服务：

```bash
ln -s /etc/httpd/sites-available/com.example /etc/httpd/sites-enabled/
```

### Apache _DocumentRoot_

您需要创建 _DocumentRoot_。使用以下命令：

```bash
mkdir -p /var/www/sub-domains/com.example/html
```

## 安装 DokuWiki

在您的服务器中，切换到根目录。

```bash
cd /root
```

获取最新稳定版本的 DokuWiki。您可以通过转到[下载页面](https://download.dokuwiki.org/)找到它。在页面左侧，在 "Version" 下，您将看到 "Stable (Recommended) (direct link)"。

右键点击 "(direct link)" 并复制链接。在 DokuWiki 服务器的控制台中，输入 `wget` 和一个空格，然后将复制的链接粘贴到终端中。您应该得到类似以下内容：

```bash
wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
```

在解压归档文件之前，使用 `tar ztf` 检查内容：

```bash
tar ztvf dokuwiki-stable.tgz
```

注意在所有其他文件前面有一个带日期的命名目录，类似于：

```text
... (上方更多内容)
dokuwiki-2020-07-29/inc/lang/fr/resetpwd.txt
dokuwiki-2020-07-29/inc/lang/fr/draft.txt
dokuwiki-2020-07-29/inc/lang/fr/recent.txt
... (下方更多内容)
```

解压归档时您不想保留那个前导的命名目录，所以使用 `tar` 选项来排除它。第一个选项是 `--strip-components=1`，它会移除前导目录。第二个选项是 `-C` 选项，告诉 `tar` 您希望归档解压到何处。解压命令将类似如下：

```bash
tar xzf dokuwiki-stable.tgz  --strip-components=1 -C /var/www/sub-domains/com.example/html/
```

运行此命令后，所有 DokuWiki 文件应该都在您的 _DocumentRoot_ 中了。

您需要复制 DokuWiki 附带的 `.htaccess.dist` 文件，并保留原文件，以防需要恢复原始设置。

在此过程中，您将其重命名为 `.htaccess`。这就是 _apache_ 将寻找的文件名。为此：

```bash
cp /var/www/sub-domains/com.example/html/.htaccess{.dist,}
```

将新目录及其文件的所有权更改为 _apache_ 用户和组：

```bash
chown -Rf apache.apache /var/www/sub-domains/com.example/html
```

## 设置 DNS 或 `/etc/hosts`

在可以访问 DokuWiki 界面之前，必须为此站点设置名称解析。您可以使用 `/etc/hosts` 文件进行测试。

在本例中，假设 DokuWiki 将在私有 IPv4 地址 10.56.233.179 上运行。假设您也在 Linux 工作站上修改 `/etc/hosts` 文件。为此，运行：

```bash
sudo vi /etc/hosts
```

然后更改您的 hosts 文件，使其看起来类似以下内容（注意前面示例中的 IP 地址）：

```bash
127.0.0.1 localhost
127.0.1.1 myworkstation-home
10.56.233.179 example.com     example

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

测试完成并准备好正式投入使用时，必须将此主机添加到 DNS 服务器中。您可以使用[私有 DNS 服务器](../dns/private_dns_server_using_bind.md)，或面向公众的 DNS 服务器。

## 启动 `httpd`

启动 `httpd` 之前，先测试确保配置正确：

```bash
httpd -t
```

您应该得到：

```text
Syntax OK
```

如果是这样，您就可以启动 `httpd`，然后完成设置了。首先启用 `httpd` 在启动时启动：

```bash
systemctl enable httpd
```

然后启动它：

```bash
systemctl start httpd
```

## 测试 DokuWiki

下一步是打开 Web 浏览器，在地址栏中输入：

<http://example.com/install.php>

这将带您进入设置界面：

- 在 "Wiki Name" 字段中，输入您的 Wiki 名称。示例 "Technical Documentation"
- 在 "Superuser" 字段中，输入管理员用户名。示例 "admin"
- 在 "Real name" 字段中，输入管理员的真实姓名
- 在 "E-Mail" 字段中，输入管理员的电子邮件地址
- 在 "Password" 字段中，输入管理员的安全密码
- 在 "once again" 字段中，重新输入相同的密码
- 在 "Initial ACL Policy" 下拉菜单中，选择最适合您环境的选项
- 选择您想要将内容置于其下的许可证的相应复选框
- 保留勾选（或根据您的偏好取消勾选）"Once a month, send anonymous usage data to the DokuWiki developers" 复选框
- 点击 "Save" 按钮

您的 Wiki 现在已准备好供您添加内容。

## 保护 DokuWiki 安全

除了刚才创建的 ACL 策略，还需要考虑以下事项：

### `firewalld` 防火墙

!!! note

    此防火墙示例不假设您可能需要在 DokuWiki 服务器上允许的其他服务。这些规则基于您的测试环境，**仅**处理允许访问本地网络 IP 块的问题。对于生产服务器，您将需要允许更多服务。

在宣告一切完成之前，您需要考虑安全性。首先，您应该在服务器上运行防火墙。

假设 10.0.0.0/8 网络上的任何人都在您的私有局域网中，并且只有这些人需要访问该站点。

如果您使用 `firewalld` 作为防火墙，请使用以下规则语法：

```bash
firewall-cmd --zone=trusted --add-source=10.0.0.0/8 --permanent
firewall-cmd --zone=trusted --add-service=http --add-service=https --permanent
firewall-cmd --reload
```

添加这些规则并重新加载 `firewalld` 服务后，列出您的区域以确保所有需要的内容都在那里：

```bash
firewall-cmd --zone=trusted --list-all
```

如果一切正常，将类似如下：

```bash
trusted (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces: 
  sources: 10.0.0.0/8
  services: http https
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```

### SSL/TLS

为了实现最佳安全性，您应考虑使用 SSL/TLS 进行加密的 Web 流量。您可以从 SSL/TLS 提供商购买 SSL/TLS 证书，或使用 [Let's Encrypt](../security/generating_ssl_keys_lets_encrypt.md)。

## 总结

无论您需要记录流程、公司政策、程序代码还是其他内容，Wiki 都是一个很好的方式。DokuWiki 是一个安全、灵活、易于使用的产品，安装和部署也很简单。它也是一个已经存在多年的稳定项目。
