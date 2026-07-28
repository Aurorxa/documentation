---
title: 使用 LXD 运行 Web 服务器
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - containers
  - lxd
  - apache
---

# LXD Web 服务器入门指南 —— 多台机器实现多站点

## 前提条件

* 一台运行 Rocky Linux 的服务器，已安装了 LXD。如果您没有 LXD，请优先使用 `incus`。请参阅此处的 [incus 文档](../../../books/incus_server/00-toc.md)
* 具备在 `vi` 或其他编辑器中进行编辑的技能
* 能够以 root 身份进入机器，或通过 `sudo` 获取 root 访问权限
* 了解基本的网络内部部署、端口转发、包过滤防火墙
* 如果您希望您的 Web 服务器可从公网访问，您需要一个注册域名和您公网 IP 地址的正确 DNS 解析，并且机器必须位于您的 DMZ（隔离区）中

## 引言

有时候，在两个容器环境中分别运行两台 Web 服务器，比使用主机系统本地的虚拟主机(Virtual Hosts)来运行两到三个不同的网站更容易、更具灵活性。使用 `incus` 或 `lxd` 容器，您可以避免使用虚拟主机，并允许您使用完全独立的资源。

您甚至可以定义一个公共 IP 地址，并有多个容器各自运行自己的 Web 服务器，由反向代理(reverse proxy)服务器管理流量分发。这样更进一步，为您提供了更大的灵活性，特别是对于处理 PHP-FPM 资源、重启 Apache 实例等场景。

这台服务器还可以充当您的 DMZ 区域内的 Web 服务器。在本文中，假设您使用的是 [LXD Web 服务器](https://docs.rockylinux.org/guides/containers/lxd_web_servers/) 设置

## 前提条件

我们假设您已参考上述文档部署了 LXD。请注意，虽然该文档提到使用的是 "Rocky Linux 8.5 容器镜像"，但实际上应选用 Rocky Linux 最新版本的镜像。

## 安装

* 安装 EPEL 以提供一些必要的工具：

    ```bash
    dnf -y install epel-release
    ```

* 安装 Apache：

    ```bash
    dnf -y install httpd
    ```

* 启用 Apache：

    ```bash
    systemctl enable httpd
    ```

## 初始化站点

* 创建站点目录。对于我们的第一个站点，这里使用 `site1`：

    ```bash
    mkdir -p /var/www/sub-domains/site1/html
    ```

* 将您站点的 Web 文件放置在上面的 `html` 目录中。

* 接下来，需要设置权限。[Apache 多站点指南](https://docs.rockylinux.org/guides/web/apache-sites-enabled/)中有相关描述。

* 另外，需要创建一个 `index.html` 文件：

    ```bash
    vi /var/www/sub-domains/site1/html/index.html
    ```

    `index.html` 可以包含您希望的任何内容，但至少要包含 "Hello World" 之类的内容，以便进行测试。

* 现在我们来创建配置文件。实际上，我们会复制一份默认配置，并根据需要修改。请将 `site1` 替换为您实际的站点名：

    ```bash
    cp /etc/httpd/conf.d/site1.conf /etc/httpd/conf.d/com.site1.conf
    ```

    !!! note

        您在 `/etc/httpd/conf.d/` 中放置的任何文件必须以 `.conf` 结尾。

* 接下来，编辑 `com.site1.conf` 并做必要的更改：

    ```bash
    vi /etc/httpd/conf.d/com.site1.conf
    ```

    示例配置：

    ```bash
    <VirtualHost *:80>
        ServerName www.site1.com
        ServerAdmin username@rockylinux.org
        DocumentRoot /var/www/sub-domains/site1/html

        <Directory /var/www/sub-domains/site1/html>
                    Options -Indexes +FollowSymLinks
                    AllowOverride All
        </Directory>

        ErrorLog /var/log/httpd/site1_public_html_error.log
        CustomLog /var/log/httpd/site1_public_html_requests.log combined
    </VirtualHost>
    ```

## 设置

要设置第一个容器，请执行以下操作：

* 列出您的容器：

    ```text
    lxc list
    ```

    （或）使用 incus：

    ```text
    incus list
    ```

    这将返回：

    ```text
    +----------+---------+----------------------+------+-----------+-----------+
    |   NAME   |  STATE  |          IPV4        | IPV6 |   TYPE    | SNAPSHOTS |
    +----------+---------+----------------------+------+-----------+-----------+
    | rockylinux | RUNNING | 10.0.0.1 (eth0) |        | CONTAINER | 0         |
    +----------+---------+----------------------+------+-----------+-----------+
    ```

* 使用 （`lxc` 或 `incus`） 进入容器：

    ```bash
    lxc exec rockylinux bash
    ```

* 然后，在容器内部启动 `httpd`：

    ```bash
    systemctl start httpd
    ```

### 通过 Web 浏览器进行测试

在本地主机上，使用指向容器 IP 地址的链接进行测试：

```text
http://10.0.0.1
```

!!! note

    如果您是从其他机器进行测试，且没有配置 DNS 服务器，您可以编辑本地 `/etc/hosts` 文件，添加一个条目指向您容器的 IP 地址和主机名。例如：

    ```text
    10.0.0.1 site1.com www.site1.com
    ```

这将验证 Web 服务器是否正常运行。下一步是配置防火墙。

## 容器防火墙

要配置防火墙，您需要先安装 `iptables`，因为容器本身不带防火墙。这有助于在测试结束后移除 `iptables`。

使用 `iptables` 禁用直接入站流量通常是个好主意。这可以防止通过搜索 IP 地址找到您的服务器。此外，您必须确保 `http` 端口（或其他任何端口）能够接受流量。首先，完全阻止端口 80：

```bash
iptables -I INPUT -p tcp --dport 80 -j DROP
```

然后允许来自特定 IP 地址的端口 80 流量。假设您需要允许 `10.0.0.1`（正如本例中使用的桥接地址）：

```bash
iptables -I INPUT -s 10.0.0.1 -p tcp --dport 80 -j ACCEPT
```

如果您只有一台 Web 服务器，这些配置可能就足够了。如果计划部署多台 Web 服务器，您可能希望调整每个容器的 `iptables` 规则，使其仅接受来自防火墙 IP 的流量。理想情况下，您可以在网络边缘部署一个反向代理，将正确的流量传送给正确的 Web 服务器。

## 在容器上配置其他站点

在容器上配置其他站点时，按照上述相同的流程操作：创建 `html` 目录、创建配置文件、设置权限，然后重启 `httpd`，或者如果 `httpd` 正在运行，执行 `systemctl reload httpd`。

## 结论

虽然您可以在同一环境中运行虚拟主机，但利用容器来隔离各个站点，往往会带来更大的灵活性。这也有助于灾难恢复，并且资源可以保持相互隔离。
