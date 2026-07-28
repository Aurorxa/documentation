---
author: Antoine Le Morvan
contributors: Ganna Zhyrnova
title: 第 5.2 部分 Varnish
---

## Varnish

本章将教您关于 Web 加速反代缓存的知识：Varnish。

****

**目标**：您将学习如何：

:heavy_check_mark: 安装和配置 Varnish；  
:heavy_check_mark: 缓存网站内容。

:checkered_flag: **reverse-proxy**, **cache**

**知识**：:star: :star:  
**复杂性**：:star: :star: :star:  

**阅读时间**：30 分钟

****

### 概述

Varnish 是一个 HTTP 反向代理缓存服务，或称网站加速器。

Varnish 接收来自访问者的 HTTP 请求：

* 如果缓存请求的响应可用，它直接从服务器内存中将响应返回给客户端，
* 如果没有响应，Varnish 则请求 Web 服务器。然后 Varnish 将请求发送到 Web 服务器，检索响应，将其存储在其缓存中，并响应客户端。

从内存缓存中响应可以改善客户端的响应时间。在这种情况下，没有对物理磁盘的访问。

默认情况下，Varnish 监听 **6081** 端口，并使用 **VCL**（**V**arnish **C**onfiguration **L**anguage，Varnish 配置语言）进行配置。借助 VCL，可以：

* 决定客户端通过传输方式接收的内容
* 确定缓存的内容
* 从哪个站点以及如何修改响应

Varnish 可通过 VMOD 模块（Varnish 模块）进行扩展。

#### 确保高可用性

使用多种机制可确保整个 Web 链的高可用性：

* 如果 Varnish 位于负载均衡器（LB）之后，则 LB 已处于 HA（高可用）模式，因为 LB 通常处于集群模式。LB 的检查验证 Varnish 的可用性。如果 Varnish 服务器不再响应，则自动将其从可用服务器池中移除。在这种情况下，Varnish 处于 ACTIVE/ACTIVE（主/主）模式。
* 如果 Varnish 不在 LB 集群之后，客户端则访问在 2 个 Varnish 之间共享的 VIP（Virtual IP Address，虚拟 IP 地址）（参见 Heartbeat 章节）。在这种情况下，Varnish 处于 ACTIVE/PASSIVE（主/备）模式。如果活动服务器不可用，VIP 将切换到第二个 Varnish 节点。
* 当后端不可用时，可以从 Varnish 后端池中移除它，可以自动（通过健康检查）或手动在 CLI 模式下（对于简化升级或更新很有用）。

#### 确保可扩展性

如果后端不再足以支持工作负载：

* 要么为后端添加更多资源并重新配置中间件
* 要么向 Varnish 后端池添加另一个后端

#### 促进可扩展性

一个 Web 页面通常由 HTML（通常由 PHP 动态生成）和更多的静态资源（JPG、gif、CSS、js 等）在创建过程中组成。缓存可缓存的资源（静态资源）很快变得有趣，这会减轻后端的许多请求。

!!! NOTE

    缓存 Web 页面（HTML、PHP、ASP、JSP 等）是可能的，但更复杂。您需要了解应用程序以及页面是否可缓存，这在使用 REST API 时应该是可行的。

当客户端直接访问 Web 服务器时，服务器必须返回与客户端请求次数相同的图像。一旦客户端首次收到图像，它会根据站点和 Web 应用程序的配置在浏览器端缓存。

当通过正确配置的缓存服务器访问服务器时，请求图像的第一个客户端将发起一个初始后端请求。但是，图像的缓存将在一定时间内发生，随后的分发将定向到请求相同资源的其他客户端。

虽然配置良好的浏览器端缓存减少了到后端的请求数量，但它补充了使用 Varnish 代理缓存的作用。

#### TLS 证书管理

Varnish 不能以 HTTPS 通信（这也不是它的职责）。

因此，证书必须由以下之一承载：

* 当流量通过 LB 时由 LB 承载（推荐的解决方案是集中管理证书等）。然后流量以未加密方式通过数据中心。
* 由 Varnish 服务器本身上的 Apache、Nginx 或 HAProxy 服务承载，该服务仅充当 Varnish 的代理（从 443 端口到 80 端口）。当直接访问 Varnish 时，此解决方案很有用。
* 同样，Varnish 不能与 443 端口上的后端通信。必要时，您需要使用 Nginx 或 Apache 反向代理来为 Varnish 解密请求。

#### 工作原理

在基本的 Web 服务中，客户端通过 80 端口的 TCP 直接与服务通信。

![标准网站的工作方式](img/varnish_website.png)

要使用缓存，客户端必须在默认的 Varnish 端口 6081 上与 Web 服务通信。

![Varnish 默认工作方式](img/varnish_website_with_varnish.png)

为了使服务对客户端透明，您必须更改 Varnish 和 Web 服务 vhost 的默认监听端口。

![对客户端透明的实现](img/varnish_website_with_varnish_port_80.png)

要提供 HTTPS 服务，可以在 Varnish 服务上游添加负载均衡器，或在 Varnish 服务器上添加代理服务，如 Apache、Nginx 或 HAProxy。

### 配置

安装非常简单：

```bash
dnf install -y varnish
systemctl enable varnish
systemctl start varnish
```

#### 配置 Varnish 守护进程

从 `systemctl` 开始，Varnish 参数通过服务文件 `/usr/lib/systemd/system/varnish.service` 设置：

```bash
[Unit]
Description=Varnish Cache, a high-performance HTTP accelerator
After=network-online.target

[Service]
Type=forking
KillMode=process

# 最大打开文件数（用于 ulimit -n）
LimitNOFILE=131072

# 锁定的共享内存——应该足以锁定共享内存日志
# （varnishd -l 参数）
# 默认日志大小为 80MB vsl + 1M vsm + 头 -> 82MB
# 单位为字节
LimitMEMLOCK=85983232

# 启用此选项以避免重新加载时出现 "fork failed"。
TasksMax=infinity

# Core 文件的最大大小。
LimitCORE=infinity

ExecStart=/usr/sbin/varnishd -a :6081 -f /etc/varnish/default.vcl -s malloc,256m
ExecReload=/usr/sbin/varnishreload

[Install]
WantedBy=multi-user.target
```

使用 `systemctl edit varnish.service` 更改默认值：这将创建 `/etc/systemd/system/varnish.service.d/override.conf` 文件：

```bash
$ sudo systemctl edit varnish.service
[Service]
ExecStart=/usr/sbin/varnishd -a :6081 -f /etc/varnish/default.vcl -s malloc,512m
```

您可以选择多次选项来指定缓存存储后端。可能的存储类型是 `malloc`（在内存中缓存，必要时交换），或 `file`（在磁盘上创建文件，然后映射到内存）。大小以 K/M/G/T（千字节、兆字节、千兆字节或太字节）表示。

#### 配置后端

Varnish 使用一种称为 VCL 的特定语言进行配置。

这涉及将 VCL 配置文件编译为 C。如果编译成功且没有警告，则可以重新启动服务。

您可以使用以下命令测试 Varnish 配置：

```bash
varnishd -C -f /etc/varnish/default.vcl
```

!!! NOTE

    建议在重新启动 `varnishd` 守护进程之前检查 VCL 语法。

使用以下命令重新加载配置：

```bash
systemctl reload varnishd
```

!!! warning

`systemctl restart varnishd` 会清空 Varnish 缓存并导致后端负载峰值。因此，应该避免重新加载 `varnishd`。

!!! NOTE

    要配置 Varnish，请遵循此页面上的建议：<https://www.getpagespeed.com/server-setup/varnish/varnish-virtual-hosts>。

### VCL 语言

#### 子例程

Varnish 使用 VCL 文件，分割为包含要运行动作的子例程。这些子例程仅在它们定义的特定情况下运行。默认的 `/etc/varnish/default.vcl` 文件包含 `vcl_recv`、`vcl_backend_response` 和 `vcl_deliver` 例程：

```bash
#
# 这是一个 Varnish 的示例 VCL 文件。
#
# 默认情况下它不执行任何操作，将控制委托给内置 VCL。
# 当没有显式的 return 语句时调用内置 VCL。
#
# 请参阅用户指南中的 VCL 章节：https://www.varnish-cache.org/docs/
# 以及 http://varnish-cache.org/trac/wiki/VCLExamples 获取更多示例。

# 告诉 VCL 编译器此 VCL 已适配新的 4.0 格式的标记。
vcl 4.0;

# 默认后端定义。将其设置为指向您的内容服务器。
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

sub vcl_recv {

}

sub vcl_backend_response {

}

sub vcl_deliver {

}
```

* **vcl_recv**：在将请求发送到后端之前调用的例程。在此例程中，您可以修改 HTTP 头和 cookie，选择后端等。参见 `set req` 操作。
* **vcl_backend_response**：在接收后端响应后调用的例程（`beresp` 表示 BackEnd RESPonse）。参见 `set bereq.` 和 `set beresp.` 操作。
* **vcl_deliver**：此例程对于修改 Varnish 输出很有用。如果您需要修改最终对象（例如，添加或移除一个头），可以在 `vcl_deliver` 中执行。

#### VCL 操作符

* `=`：赋值
* `==`：比较
* `~`：与正则表达式和 ACL 组合比较
* `!`：否定
* `&&`：逻辑与
* `||`：逻辑或

#### Varnish 对象

* **req**：请求对象。当 Varnish 收到请求时创建 `req`。`vcl_recv` 子例程中的大部分工作都与此对象相关。
* **bereq**：发往 Web 服务器的请求对象。Varnish 从 `req` 创建此对象。
* **beresp**：Web 服务器响应对象。它包含来自应用程序的对象头。您可以在 `vcl_backend_response` 子例程中修改服务器响应。
* **resp**：发送给客户端的 HTTP 响应。使用 `vcl_deliver` 子例程修改此对象。
* **obj**：缓存的对象。只读。

#### Varnish 操作

最常见的操作：

* **pass**：返回时，请求和后续响应将来自应用服务器。不应用缓存。`pass` 从 `vcl_recv` 子例程返回。
* **hash**：当从 `vcl_recv` 返回时，Varnish 将从缓存中提供内容，即使请求的配置指定不经缓存传递。
* **pipe**：用于管理流。在这种情况下，Varnish 将不再检查每个请求，而是将所有字节传递到服务器。例如，Websocket 或视频流管理使用 `pipe`。
* **deliver**：将对象传递给客户端。通常从 `vcl_backend_response` 子例程。
* **restart**：重新启动请求处理过程。保留对 `req` 对象的修改。
* **retry**：将请求重新传输回应用服务器。如果应用响应不令人满意，从 `vcl_backend_response` 或 `vcl_backend_error` 使用。

总结起来，下面图表说明了子例程和操作之间可能的交互：

![对客户端透明的实现](img/varnish_interactions.png)

### 验证/测试/故障排除

可以从 HTTP 响应头验证页面是否来自 Varnish 缓存：

![简化的 Varnish 操作](img/varnish_troobleshooting.png)

### 后端

Varnish 使用术语 `backend`（后端）表示它需要代理的 vhost。

您可以在同一个 Varnish 服务器上定义多个后端。

通过 `/etc/varnish/default.vcl` 配置后端。

#### ACL 管理

```bash
# 拒绝 ACL
acl deny {
"10.10.0.10"/32;
"192.168.1.0"/24;
}
```

应用 ACL：

```bash
# 阻止 ACL 拒绝的 IP
if (client.ip ~ forbidden) {
  error 403 "Access forbidden";
}
```

不缓存某些页面：

```bash
# 不缓存登录和管理页面
if (req.url ~ "/(login|admin)") {
  return (pass);
}
```

#### POST 和 Cookie 设置

Varnish 从不缓存 HTTP POST 请求或包含 cookie 的请求（无论是来自客户端还是后端）。

如果后端使用 cookie，内容缓存将不会发生。

要纠正此行为，您可以在请求中取消设置 cookie：

```bash
sub vcl_recv {
    unset req.http.cookie;
}

sub vcl_backend_response {
    unset beresp.http.set-cookie;
}
```

#### 将请求分发到不同的后端

当托管多个站点时，例如一个文档服务器（<docs.rockylinux.org>）和一个 wiki（<wiki.rockylinux.org>），可以将请求分发到正确的后端。

后端声明：

```bash
backend docs {
    .host = "127.0.0.1";
    .port = "8080";
}

backend blog {
    .host = "127.0.0.1";
    .port = "8081";
}
```

根据 HTTP 请求中调用的主机，在 `vcl_recv` 子例程中修改 `req.backend` 对象：

```bash
sub vcl_recv {
    if (req.http.host ~ "^doc.rockylinux.org$") {
        set req.backend = docs;
    }

    if (req.http.host ~ "^wiki.rockylinux.org$") {
        set req.backend = wiki;
    }
}
```

#### 负载分发

Varnish 可以使用称为 director 的特定后端处理负载均衡。

round-robin director 以轮询方式（交替地）将请求分发到后端。您可以为每个后端分配权重。

client director 根据任何头元素上的粘性会话关联性（即，使用会话 cookie）分发请求。在这种情况下，客户端总是被返回到同一个后端。

后端声明：

```bash
backend docs1 {
    .host = "192.168.1.10";
    .port = "8080";
}

backend docs2 {
    .host = "192.168.1.11";
    .port = "8080";
}
```

`director` 允许您关联 2 个定义的后端。

Director 声明：

```bash
director docs_director round-robin {
    { .backend = docs1; }
    { .backend = docs2; }
}
```

剩下要做的就是将 director 定义为请求的后端：

```bash
sub vcl_recv {
    set req.backend = docs_director;
}
```

#### 使用 CLI 管理后端

出于管理或维护目的，可以将后端标记为 **sick**（故障）或 **healthy**（健康）。此操作允许您从池中移除节点，而无需修改 Varnish 服务器配置（无需重启）或停止后端服务。

查看后端状态：`backend.list` 命令显示所有后端，即使那些没有健康检查（probe）的。

```bash
$ varnishadm backend.list
Backend name                   Admin      Probe
site.default                   probe      Healthy (no probe)
site.front01                   probe      Healthy 5/5
site.front02                   probe      Healthy 5/5
```

切换状态：

```bash
varnishadm backend.set_health site.front01 sick

varnishadm backend.list
Backend name                   Admin      Probe
site.default                   probe      Healthy (no probe)
site.front01                   sick       Sick 0/5
site.front02                   probe      Healthy 5/5

varnishadm backend.set_health site.front01 healthy

varnishadm backend.list
Backend name                   Admin      Probe
site.default                   probe      Healthy (no probe)
site.front01                   probe      Healthy 5/5
site.front02                   probe      Healthy 5/5
```

要让 Varnish 决定其后端的状态，必须手动将后端切换为 sick 或 healthy 后端，再切换回 auto 模式。

```bash
varnishadm backend.set_health site.front01 auto
```

后端的声明按照以下方式：<https://github.com/mattiasgeniar/varnish-6.0-configuration-templates>。

### Apache 日志

由于 HTTP 服务被反向代理，Web 服务器将不再能访问客户端的 IP 地址，而是只能访问 Varnish 服务。

要在 Apache 日志中考虑反向代理，请更改服务器配置文件中的事件日志格式：

```bash
LogFormat "%{X-Forwarded-For}i %l %u %t "%r" %>s %b "%{Referer}i" "%{User-Agent}i"" varnishcombined
```

并在网站 vhost 中使用此新格式：

```bash
CustomLog /var/log/httpd/www-access.log.formatux.fr varnishcombined
```

并使其兼容 Varnish：

```bash
if (req.restarts == 0) {
  if (req.http.x-forwarded-for) {
    set req.http.X-Forwarded-For = req.http.X-Forwarded-For + ", " + client.ip;
  } else {
   set req.http.X-Forwarded-For = client.ip;
  }
}
```

### 缓存清除

一些清除缓存的请求：

在命令行上：

```bash
varnishadm 'ban req.url ~ .'
```

使用 secret 和非默认端口：

```bash
varnishadm -S /etc/varnish/secret -T 127.0.0.1:6082 'ban req.url ~ .'
```

在 CLI 上：

```bash
varnishadm

varnish> ban req.url ~ ".css$"
200

varnish> ban req.http.host == example.com
200

varnish> ban req.http.host ~ .
200
```

通过 HTTP PURGE 请求：

```bash
curl -X PURGE http://example.com/foo.txt
```

配置 Varnish 接受此请求的方法如下：

```bash
acl local {
    "localhost";
    "10.10.1.50";
}

sub vcl_recv {
    # 指令应放在首位，
    # 否则其他指令可能先匹配
    # 而清除永远不会执行
    if (req.method == "PURGE") {
        if (client.ip ~ local) {
            return(purge);
        }
    }
}
```

### 日志管理

Varnish 将其日志写入内存并以二进制形式记录，以不损害其性能。当内存空间不足时，它会将新记录覆盖在旧记录之上，从其内存空间的开头开始。

可以使用 `varnishstat`（统计信息）、`varnishtop`（Varnish 的 top）、`varnishlog`（详细日志）或 `varnishnsca`（NCSA 格式日志，类似 Apache）工具查阅日志：

```bash
varnishstat
varnishtop -i ReqURL
varnishlog
varnishnsca
```

使用 `-q` 选项对日志应用过滤：

```bash
varnishlog -q 'TxHeader eq MISS' -q "ReqHeader ~ '^Host: rockylinux\.org$'"
varnishncsa -q "ReqHeader eq 'X-Cache: MISS'"
```

`varnishlog` 和 `varnishnsca` 守护进程独立于 `varnishd` 守护进程将日志写入磁盘。`varnishd` 守护进程继续在内存中填充其日志，而不会影响客户端的性能；然后，其他守护进程将日志复制到磁盘。

### 实践坊

对于本实践坊，您需要一台服务器，已按前面章节的描述安装、配置和保护了 Apache 服务。

您将在其前配置一个反向代理缓存。

您的服务器具有以下 IP 地址：

* server1: 192.168.1.10

如果您没有用于解析名称的服务，请按如下方式填充 `/etc/hosts` 文件：

```bash
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.1.10 server1 server1.rockylinux.lan
```

#### 任务 1：安装和配置 Apache

```bash
sudo dnf install -y httpd mod_ssl
sudo systemctl enable httpd  --now
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
echo "<html><body>Node $(hostname -f)</body></html>" | sudo tee "/var/www/html/index.html"
```

验证：

```bash
$ curl http://server1.rockylinux.lan
<html><body>Node server1.rockylinux.lan</body></html>

$ curl -I http://server1.rockylinux.lan
HTTP/1.1 200 OK
Date: Mon, 12 Aug 2024 13:16:18 GMT
Server: Apache/2.4.57 (Rocky Linux) OpenSSL/3.0.7
Last-Modified: Mon, 12 Aug 2024 13:11:54 GMT
ETag: "36-61f7c3ca9f29c"
Accept-Ranges: bytes
Content-Length: 54
Content-Type: text/html; charset=UTF-8
```

#### 任务 2：安装 Varnish

```bash
sudo dnf install -y varnish
sudo systemctl enable varnishd --now
sudo firewall-cmd --permanent --add-port=6081/tcp --permanent
sudo firewall-cmd --reload
```

#### 任务 3：将 Apache 配置为后端

修改 `/etc/varnish/default.vcl` 以使用 Apache（80 端口）作为后端：

```bash
# 默认后端定义。将其设置为指向您的内容服务器。
backend default {
    .host = "127.0.0.1";
    .port = "80";
}
```

重新加载 Varnish

```bash
sudo systemctl reload varnish
```

检查 Varnish 是否工作：

```bash
$ curl -I http://server1.rockylinux.lan:6081
HTTP/1.1 200 OK
Server: Apache/2.4.57 (Rocky Linux) OpenSSL/3.0.7
X-Varnish: 32770 6
Age: 8
Via: 1.1 varnish (Varnish/6.6)

$ curl http://server1.rockylinux.lan:6081
<html><body>Node server1.rockylinux.lan</body></html>
```

如您所见，Apache 提供了索引页面。

添加了一些头，向我们提供信息，表明我们的请求是由 Varnish 处理的（头 `Via`）和页面的缓存时间（头 `Age`），这告诉我们我们的页面直接从 Varnish 内存中提供，而不是从 Apache 的磁盘中提供。

#### 任务 4：移除一些头

我们将移除一些可能为黑客提供不必要信息的头。

在 `vcl_deliver` 子例程中，添加以下内容：

```bash
sub vcl_deliver {
    unset resp.http.Server;
    unset resp.http.X-Varnish;
    unset resp.http.Via;
    set resp.http.node = "F01";
    set resp.http.X-Cache-Hits = obj.hits;
    if (obj.hits > 0) { # 添加调试头以查看是否是 HIT/MISS 和命中次数，不需要时禁用
      set resp.http.X-Cache = "HIT";
    } else {
      set resp.http.X-Cache = "MISS";
    }
}
```

测试您的配置并重新加载 Varnish：

```bash
$ sudo varnishd -C -f /etc/varnish/default.vcl
...
$ sudo systemctl reload varnish
```

检查差异：

```bash
$ curl -I http://server1.rockylinux.lan:6081
HTTP/1.1 200 OK
Age: 4
node: F01
X-Cache-Hits: 1
X-Cache: HIT
Accept-Ranges: bytes
Connection: keep-alive
```

如您所见，移除了不需要的头，同时添加了必要的头（用于故障排除）。

### 总结

您现在拥有设置主缓存服务器和添加功能所需的所有知识。

在基础设施中拥有一个 Varnish 服务器除了缓存之外，还可以做很多有用的事情：用于后端服务器安全、处理头信息、简化更新（例如蓝/绿或金丝雀模式）等。

### 知识自测

:heavy_check_mark: Varnish 可以托管静态文件吗？

* [ ] 正确  
* [ ] 错误  

:heavy_check_mark: Varnish 缓存必须存储在内存中吗？

* [ ] 正确  
* [ ] 错误  
