---
title: HAProxy-Apache-LXD
author: Steven Spencer
contributors: Ezequiel Bruni, Antoine Le Morvan, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
---

# 使用 LXD 容器实现 HAProxy 负载均衡 Apache

## 简介

HAProxy 代表"High Availability Proxy"（高可用代理）。此代理可以位于任何 TCP 应用程序（如 Web 服务器）之前，但它通常用作多个网站实例之间的负载均衡器。

这样做有很多理由。如果您的网站访问量很大——添加同一网站的另一个实例并将 HAProxy 置于两者之前——可以让您在实例之间分配流量。另一个理由可能是能够在没有任何停机时间的情况下更新网站内容。HAProxy 还可以帮助缓解 DOS 和 DDOS 攻击。

本指南将探讨在同一台 LXD 主机上使用 HAProxy 配合两个网站实例，以及使用轮询（round-robin）轮换进行负载均衡。这可能是确保可以在不停机的情况下执行更新的一个完美解决方案。

但是，如果您的问题是网站性能，您可能需要将多个站点分布到实际的裸机或多台 LXD 主机上。当然也可以在不使用 LXD 的情况下在裸机上完成所有这些操作。但是，LXD 提供了极大的灵活性和性能，而且也非常适合实验室测试。

## 前提条件与假设

- 在 Linux 机器上完全熟悉命令行
- 具有命令行编辑器使用经验（此处使用 `vim`）
- 具有 `crontab` 使用经验
- 了解 LXD。有关更多信息，您可能需要查阅 [LXD Server](../../books/lxd_server/00-toc.md) 文档。在笔记本或工作站上安装 LXD 而无需进行完整的服务器安装是可以的。本文档使用运行 LXD 的实验机器编写，但并未设置为上述链接文档中使用的完整服务器。
- 具备一些安装、配置和使用 Web 服务器的知识。
- 我们假设 LXD 已安装并准备好创建容器。

## 安装容器

在本指南的 LXD 主机上，您需要三个容器。如果您愿意，可以有更多的 Web 服务器容器。您将使用 **web1** 和 **web2** 作为我们的网站容器，使用 **proxyha** 作为我们的 HAProxy 容器。要在您的 LXD 主机上安装这些容器，请执行：

```bash
lxc launch images:rockylinux/8 web1
lxc launch images:rockylinux/8 web2
lxc launch images:rockylinux/8 proxyha
```

运行 `lxc list` 应该返回类似以下内容：

```bash
+---------+---------+----------------------+------+-----------+-----------+
|  NAME   |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+---------+---------+----------------------+------+-----------+-----------+
| proxyha | RUNNING | 10.181.87.137 (eth0) |      | CONTAINER | 0         |
+---------+---------+----------------------+------+-----------+-----------+
| web1    | RUNNING | 10.181.87.207 (eth0) |      | CONTAINER | 0         |
+---------+---------+----------------------+------+-----------+-----------+
| web2    | RUNNING | 10.181.87.34 (eth0)  |      | CONTAINER | 0         |
+---------+---------+----------------------+------+-----------+-----------+
```

## 创建和使用 `macvlan` 配置文件

容器在默认的桥接接口上运行，使用桥接分配的 DHCP 地址。这些需要更改为来自本地 LAN 的 DHCP 地址。首先需要创建并分配 `macvlan` 配置文件。

首先创建配置文件：

`lxc profile create macvlan`

确保您的编辑器设置为您偏好的编辑器，本例中为 `vim`：

`export EDITOR=/usr/bin/vim`

接下来，更改 `macvlan` 配置文件。在此之前，您需要知道主机用于 LAN 的接口。运行 `ip addr` 并查找具有 LAN IP 分配的接口：

```bash
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether a8:5e:45:52:f8:b6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.141/24 brd 192.168.1.255 scope global dynamic noprefixroute eno1
```

!!! Note

    在这种情况下，您要查找的接口是 "eno1"，这在您的系统上可能完全不同。请使用**您自己的**接口信息！

现在您知道了 LAN 接口，可以更改我们的 `macvlan` 配置文件。为此，在命令行中输入：

`lxc profile edit macvlan`

编辑配置文件使其看起来像这样。作者排除了文件顶部的注释，但如果您是 LXD 新手，请查看那些注释：

```bash
config: {}
description: ""
devices:
  eth0:
    name: eth0
    nictype: macvlan
    parent: eno1
    type: nic
name: macvlan
```

创建 `macvlan` 配置文件时，系统会复制 `default` 配置文件。更改 `default` 配置文件是不可能的。

现在 `macvlan` 配置文件已存在，您需要将其应用到我们的三个容器：

```bash
lxc profile assign web1 default,macvlan
lxc profile assign web2 default,macvlan
lxc profile assign proxyha default,macvlan
```

不幸的是，内核中实现的 `macvlan` 默认行为在 LXD 容器中莫名其妙地存在问题（参见[此文档](../../books/lxd_server/06-profiles.md)），需要在每个容器中通过 `dhclient` 在启动时获取 IP。

使用 DHCP 时操作相当简单。只需按以下步骤对每个容器操作：

- `lxc exec web1 bash` 这将使您进入 **web1** 容器的命令行
- `crontab -e` 这将编辑容器上 root 的 `crontab`
- 输入 ++i++ 进入插入模式。
- 添加一行：`@reboot /usr/sbin/dhclient`
- 按 ++escape++ 键退出插入模式。
- 用 ++shift++ + ++:++ + ++w++ + ++q++ 保存更改
- 输入 `exit` 退出容器

对 **web2** 和 **proxyha** 重复这些步骤。

完成这些步骤后，重启容器：

```bash
lxc restart web1
lxc restart web2
lxc restart proxyha
```

当您再次执行 `lxc list` 时，您将看到 DHCP 地址现在是从您的 LAN 分配的：

```bash
+---------+---------+----------------------+------+-----------+-----------+
|  NAME   |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+---------+---------+----------------------+------+-----------+-----------+
| proxyha | RUNNING | 192.168.1.149 (eth0) |      | CONTAINER | 0         |
+---------+---------+----------------------+------+-----------+-----------+
| web1    | RUNNING | 192.168.1.150 (eth0) |      | CONTAINER | 0         |
+---------+---------+----------------------+------+-----------+-----------+
| web2    | RUNNING | 192.168.1.101 (eth0) |      | CONTAINER | 0         |
+---------+---------+----------------------+------+-----------+-----------+
```

## 安装 Apache 并更改欢迎页面

我们的环境已准备就绪。接下来，在每个 Web 容器上安装 Apache (`httpd`)。您无需实际访问它们即可完成此操作：

```bash
lxc exec web1 dnf install httpd
lxc exec web2 dnf install httpd
```

对于任何现代 Web 服务器，您需要的不仅仅是 Apache，但这足以运行一些测试。

接下来，启用 `httpd`，启动它，并更改默认欢迎页面。这样，当尝试通过代理访问时，您就知道服务器正在响应。

启用并启动 `httpd`：

```bash
lxc exec web1 systemctl enable httpd
lxc exec web1 systemctl start httpd
lxc exec web2 systemctl enable httpd
lxc exec web2 systemctl start httpd
```

更改欢迎页面。此页面在没有配置网站时出现，本质上是一个加载的默认页面。在 Rocky Linux 中，此页面位于 `/usr/share/httpd/noindex/index.html`。更改该文件无需直接访问容器。只需执行以下操作：

`lxc exec web1 vi /usr/share/httpd/noindex/index.html`

搜索 `<h1>` 标签，将显示：

`<h1>HTTP Server <strong>Test Page</strong></h1>`

将该行更改为：

`<h1>SITE1 HTTP Server <strong>Test Page</strong></h1>`

对 web2 重复此过程。现在在浏览器中通过 IP 访问这些机器将返回各自的正确欢迎页面。Web 服务器还有更多工作要做，但现在先离开它们，转到代理服务器。

## 在 proxyha 上安装 HAProxy 并配置 LXD 代理

在代理容器上安装 HAProxy 很简单。同样，无需直接访问该容器：

`lxc exec proxyha dnf install haproxy`

接下来，您需要配置 `haproxy` 在端口 80 和端口 443 上监听 Web 服务。使用 `lxc` 的 configure 子命令执行此操作：

```bash
lxc config device add proxyha http proxy listen=tcp:0.0.0.0:80 connect=tcp:127.0.0.1:80
lxc config device add proxyha https proxy listen=tcp:0.0.0.0:443 connect=tcp:127.0.0.1:443
```

对于我们的测试，您将只使用端口 80 或 HTTP 流量，但这向您展示了如何配置容器在 HTTP 和 HTTPS 的默认 Web 端口上监听。使用此命令还可以确保重启 **proxyha** 容器时将保持这些监听端口。

## HAProxy 配置

您已经在容器上安装了 HAProxy，但还没有对配置做任何操作。在配置之前，您需要做一些事情来解析您的主机。通常您会使用完全限定的域名，但在此实验室环境中，您使用 IP。为了给机器关联一些名称，您将向 **proxyha** 容器添加一些主机文件记录。

`lxc exec proxyha vi /etc/hosts`

将以下记录添加到文件底部：

```bash
192.168.1.150   site1.testdomain.com     site1
192.168.1.101   site2.testdomain.com     site2
```

这使得 **proxyha** 容器能够解析这些名称。

编辑 `haproxy.cfg` 文件。您不会使用原始文件的大部分内容。您需要先将文件移动到不同的名称来备份：

`lxc exec proxyha mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.orig`

创建新的配置文件：

`lxc exec proxyha vi /etc/haproxy/haproxy.cfg`

请注意，目前 HTTPS 协议行已被注释掉。在生产环境中，您会希望使用覆盖您 Web 服务器的通配符证书并启用 HTTPS：

```bash
global
log /dev/log local0
log /dev/log local1 notice
chroot /var/lib/haproxy
stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
stats timeout 30s
user haproxy
group haproxy
daemon

# For now, all https is remarked out
#
#ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11 no-tls-tickets
#ssl-default-bind-ciphers EECDH+AESGCM:EDH+AESGCM
#tune.ssl.default-dh-param 2048

defaults
log global
mode http
option httplog
option dontlognull
option forwardfor
option http-server-close
timeout connect 5000
timeout client 50000
timeout server 50000
errorfile 400 /etc/haproxy/errors/400.http
errorfile 403 /etc/haproxy/errors/403.http
errorfile 408 /etc/haproxy/errors/408.http
errorfile 500 /etc/haproxy/errors/500.http
errorfile 502 /etc/haproxy/errors/502.http
errorfile 503 /etc/haproxy/errors/503.http
errorfile 504 /etc/haproxy/errors/504.http

# For now, all https is remarked out
# frontend www-https
# bind *:443 ssl crt /etc/letsencrypt/live/example.com/example.com.pem
# reqadd X-Forwarded-Proto:\ https

# acl host_web1 hdr(host) -i site1.testdomain.com
# acl host_web2 hdr(host) -i site2.testdomain.com

# use_backend subdomain1 if host_web1
# use_backend subdomain2 if host_web2

frontend http_frontend
bind *:80

acl web_host1 hdr(host) -i site1.testdomain.com
acl web_host2 hdr(host) -i site2.testdomain.com

use_backend subdomain1 if web_host1
use_backend subdomain2 if web_host2

backend subdomain1
# balance leastconn
  balance roundrobin
  http-request set-header X-Client-IP %[src]
# redirect scheme https if !{ ssl_fc }
     server site1 site1.testdomain.com:80 check
     server site2 web2.testdomain.com:80 check

backend subdomain2
# balance leastconn
  balance roundrobin
  http-request set-header X-Client-IP %[src]
# redirect scheme https if !{ ssl_fc }
     server site2 site2.testdomain.com:80 check
     server site1 site1.testdomain.com:80 check
```

对以上内容的简要说明。您应该在本指南的测试部分（下文）中看到这种情况：

**site1** 和 **site2** 的定义位于 "acl" 部分中。每个站点都在对方后端的轮询列表中。当您在测试中访问 site1.testdomain.com 时，URL 不会更改，但每次访问页面时，内部的页面会在 **site1** 和 **site2** 测试页面之间切换。site2.testdomain.com 也是如此。

这样做可以让您看到切换正在发生，但实际上，无论您访问哪个服务器，您的网站内容看起来都完全相同。请注意，该文档展示了您可能希望如何在多个主机之间分配流量。您也可以在 balance 行中使用 "leastconn"，这样它将加载连接数最少的站点，而不是根据上次访问进行切换。

### 错误文件

某些版本的 HAProxy 附带一组标准的 Web 错误文件，但是来自 Rocky Linux（以及上游供应商）的版本没有这些文件。您可能**确实**想要创建它们，因为它们可能帮助您排查任何问题。这些文件放在目录 `/etc/haproxy/errors` 中，该目录尚不存在。

首先，创建该目录：

`lxc exec proxyha mkdir /etc/haproxy/errors`

在该目录中创建以下每个文件。注意，您可以使用命令 `lxc exec proxyha vi /etc/haproxy/errors/filename.http` 从 LXD 主机创建每个文件名，其中 "filename.http" 指的是以下文件名之一。在生产环境中，您的公司可能有想要使用的更具体的错误页面：

文件名 `400.http`：

```bash
HTTP/1.0 400 Bad request
Cache-Control: no-cache
Connection: close
Content-Type: text/html

<html><body><h1>400 Bad request</h1>
Your browser sent an invalid request.
</body></html>
```

文件名 `403.http`：

```bash
HTTP/1.0 403 Forbidden
Cache-Control: no-cache
Connection: close
Content-Type: text/html

<html><body><h1>403 Forbidden</h1>
Request forbidden by administrative rules.
</body></html>
```

文件名 `408.http`：

```bash
HTTP/1.0 408 Request Time-out
Cache-Control: no-cache
Connection: close
Content-Type: text/html

<html><body><h1>408 Request Time-out</h1>
Your browser didn't send a complete request in time.
</body></html>
```

文件名 `500.http`：

```bash
HTTP/1.0 500 Internal Server Error
Cache-Control: no-cache
Connection: close
Content-Type: text/html

<html><body><h1>500 Internal Server Error</h1>
An internal server error occurred.
</body></html>
```

文件名 `502.http`：

```bash
HTTP/1.0 502 Bad Gateway
Cache-Control: no-cache
Connection: close
Content-Type: text/html

<html><body><h1>502 Bad Gateway</h1>
The server returned an invalid or incomplete response.
</body></html>
```

文件名 `503.http`：

```bash
HTTP/1.0 503 Service Unavailable
Cache-Control: no-cache
Connection: close
Content-Type: text/html

<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
```

文件名 `504.http`：

```bash
HTTP/1.0 504 Gateway Time-out
Cache-Control: no-cache
Connection: close
Content-Type: text/html

<html><body><h1>504 Gateway Time-out</h1>
The server didn't respond in time.
</body></html>
```

## 运行代理

在启动服务之前，为 `haproxy` 创建一个 "run" 目录：

`lxc exec proxyha mkdir /run/haproxy`

接下来，启用并启动服务：

```bash
lxc exec proxyha systemctl enable haproxy
lxc exec proxyha systemctl start haproxy
```

如果遇到任何错误，使用以下命令调查原因：

`lxc exec proxyha systemctl status haproxy`

如果一切正常启动并运行没有问题，您可以继续进行测试。

## 测试代理

与用于使我们的 **proxyha** 容器能够解析 Web 服务器的主机设置（`/etc/hosts`）类似，并且由于我们的实验室环境没有运行本地 DNS 服务器，我们在本地机器上为每个网站设置 IP 值，以对应我们的 haproxy 容器。

为此，更改您本地机器上的 `/etc/hosts` 文件。将这种域名解析方法视为一种"穷人版 DNS"。

`sudo vi /etc/hosts`

添加这两行：

```bash
192.168.1.149   site1.testdomain.com     site1
192.168.1.149   site2.testdomain.com     site2
```

如果您现在在本地机器上 ping **site1** 或 **site2**，您将收到来自 **proxyha** 的响应：

```bash
PING site1.testdomain.com (192.168.1.149) 56(84) bytes of data.
64 bytes from site1.testdomain.com (192.168.1.149): icmp_seq=1 ttl=64 time=0.427 ms
64 bytes from site1.testdomain.com (192.168.1.149): icmp_seq=2 ttl=64 time=0.430 ms
```

打开您的 Web 浏览器，在地址栏中输入 site1.testdomain.com（或 site2.testdomain.com）作为 URL。您将从两个测试页面之一收到响应，如果您再次加载页面，您将获得下一个服务器的测试页面。请注意，URL 不会更改，但返回的页面将在服务器之间交替更改。

![加载 web1 并显示第二个服务器测试消息的截图](../images/haproxy_apache_lxd.png)

## 日志记录

尽管我们的配置文件已正确设置用于日志记录，但您需要两样东西：首先，在 /var/lib/haproxy/ 中创建一个名为 "dev" 的目录：

`lxc exec proxyha mkdir /var/lib/haproxy/dev`

接下来，为 `rsyslogd` 创建一个系统进程，以从套接字（本例中为 `/var/lib/haproxy/dev/log`）获取实例并将其存储在 `/var/log/haproxy.log` 中：

`lxc exec proxyha vi /etc/rsyslog.d/99-haproxy.conf`

向该文件添加以下内容：

```bash
$AddUnixListenSocket /var/lib/haproxy/dev/log

# Send HAProxy messages to a dedicated logfile
:programname, startswith, "haproxy" {
  /var/log/haproxy.log
  stop
}
```

保存文件并退出，然后重启 `rsyslog`：

`lxc exec proxyha systemctl restart rsyslog`

要立即为该日志文件填充一些内容，再次重启 `haproxy`：

`lxc exec proxyha systemctl restart haproxy`

查看创建的日志文件：

`lxc exec proxyha more /var/log/haproxy.log`

将显示类似以下的内容：

```bash
Sep 25 23:18:02 proxyha haproxy[4602]: Proxy http_frontend started.
Sep 25 23:18:02 proxyha haproxy[4602]: Proxy http_frontend started.
Sep 25 23:18:02 proxyha haproxy[4602]: Proxy subdomain1 started.
Sep 25 23:18:02 proxyha haproxy[4602]: Proxy subdomain1 started.
Sep 25 23:18:02 proxyha haproxy[4602]: Proxy subdomain2 started.
Sep 25 23:18:02 proxyha haproxy[4602]: Proxy subdomain2 started.
```

## 结论

HAProxy 是一个功能强大的代理引擎，用于多种用途。它是一个高性能、开源的 TCP 和 HTTP 应用程序负载均衡器和反向代理。本文档演示了如何对两个 Web 服务器实例进行负载均衡。

它也可以用于其他应用程序，包括数据库。它在 LXD 容器内和独立服务器上都能工作。

还有许多本文件未涵盖的用途。请查看 [HAProxy 官方手册。](https://cbonte.github.io/haproxy-dconv/1.8/configuration.html)
