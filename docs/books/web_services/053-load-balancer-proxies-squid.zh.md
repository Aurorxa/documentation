---
author: Antoine Le Morvan
contributors: Ganna Zhyrnova
title: 第 5.3 部分 Squid
tags:
  - squid
  - proxy
  - http
---

## Squid

本章将教您关于 Squid 的知识，即 HTTP 代理缓存。

****

**目标**：您将学习如何：

:heavy_check_mark: 安装 Squid  
:heavy_check_mark: 将其配置为代理并缓存 HTTP 内容。  

:checkered_flag: **squid**, **proxy**, **HTTP**

**知识**：:star: :star:  
**复杂性**：:star: :star:  

**阅读时间**：20 分钟

****

### 概述

设置代理服务器涉及在两种架构类型之间做出选择：

* 标准代理（Standard Proxy）架构，需要为每个客户端及其 Web 浏览器进行特定配置
* 强制代理（Captive Proxy）架构，涉及拦截客户端发送的帧并将其重写为代理服务器

无论哪种情况，都会发生网络中断：客户端不能再直接物理寻址远程服务器，而必须通过代理服务器。

两个防火墙保护客户端工作站，但从不直接与外部网络通信。

![基于代理的架构](img/squid-architecture-proxy.png)

!!! Note

    此架构需要在客户端工作站上进行浏览器配置。

使用强制代理时，您不需要配置所有客户端工作站。

配置发生在网关级别，它接收客户端请求并透明地重写帧以将其发送到代理。

![基于强制代理的架构](img/squid-architecture-proxy-captif.png)

!!! Note

    此架构需要在路由器上进行特定配置。

在标准代理或强制代理架构的情况下，此类服务的主要兴趣之一是充当缓存。

这样，首次从 WAN（可能比 LAN 链路慢）下载的文件将自己存储在内存中的代理缓存中，供后续客户端使用。这优化了慢链路上的带宽。

正如您稍后将看到的，代理还有其他用途。

部署代理可以：

* 基于各种参数拒绝对特定资源的访问
* 建立客户端互联网活动的认证和监控
* 建立分布式缓存层次结构
* 从 WAN 角度隐藏 LAN 架构（LAN 上有多少客户端？）

其中优势包括：

* 互联网匿名
* 认证
* 客户端活动日志记录
* 过滤
* 限制访问
* 带宽优化
* 安全性

!!! Note

    实施认证可以阻止 LAN 上病毒的许多恶意影响。

!!! Warning

    代理服务成为需要高可用性的关键服务。

在运行 Squid 代理服务器时，管理员必须利用日志。因此，了解主要的 HTTP 响应码至关重要。

| 代码 | 类别             |
|------|------------------------|
| 1XX  | 信息                   |
| 2XX  | 成功                |
| 3XX  | 重定向            |
| 4XX  | 客户端请求错误 |
| 5XX  | 服务器错误           |

示例：

* 200：ok
* 301：Moved Permanently
* 302：Moved Temporarily
* 304：Not modified
* 400：Bad request
* 401：Unauthorized
* 404：Not found

#### 关于 Squid

Squid 支持 HTTP 和 FTP 协议。

安装基于 Squid 服务器解决方案的优势：

* 硬件解决方案昂贵
* 自 1996 年起开发
* 以 GNU/GPL 许可证发布

##### 规模规划

* 确保高可用性
* 使用快速硬盘进行缓存
* RAM 和 CPU 应正确规划

!!! Note

    每 GB 磁盘缓存分配 14MB RAM。

### 安装

安装 `squid` 包：

```bash
sudo dnf install squid
```

!!! Warning

    注意在缓存初始化之前不要启动服务！

#### Squid 服务器目录树和文件

单一配置文件是 `/etc/squid/squid.conf`。

服务日志（停止和重启）位于 `/var/log/squid.cache.log`，而客户端请求位于 `/var/log/squid/access.log`。默认情况下，缓存文件位于 `/var/spool/squid/`。

#### `squid` 命令

`squid` 命令控制 Squid 服务器。

命令语法：

```bash
squid [-z|-s|-k parse|-k rotate]
```

| 选项      | 描述                  |
|-------------|------------------------------|
| `-z`        | 初始化缓存目录 |
| `-s`        | 启用 syslog 日志记录       |
| `-k parse`  | 测试配置文件      |
| `-k rotate` | 轮转日志                 |

记录客户端请求可能很快导致存储大量数据。

定期创建新的日志文件并以压缩格式归档旧文件是一个好习惯。

您可以手动执行此操作，使用 `squid` 命令的 `-k rotate` 选项，或通过专用的 Linux 服务 `logrotate` 自动执行。

### 配置

在 `/etc/squid/squid.conf` 中配置 Squid。

* 代理端口号（监听端口）`http_port`

```bash
http_port num_port
```

!!! Note

    端口号默认设置为 3128，但经常更改为 8080。记得开放相应的防火墙端口！

当服务重启时，Squid 服务器将监听 `http_port` 指令定义的端口。

* RAM 预留 `cache_mem`

```bash
cache_mem taille KB|taille MB|taille GB
```

例如：

```bash
cache_mem 1 GB
```

!!! Tip

    最佳实践：分配总 RAM 的 1/3

* 互联网缓存协议（ICP）`icp_port`

Internet Cache Protocol（ICP，互联网缓存协议）使相邻的 Squid 服务器能够交换请求。通常的做法是提出一个共享其信息库的代理层次结构。

`icp_port` 指令定义了 Squid 用于发送和接收来自相邻 Squid 服务器的 ICP 请求的端口号。

!!! Tip

    设置为 0 以停用。

* 匿名 FTP 用户 `ftp_user`

`ftp_user` 指令将 FTP 用户与匿名 FTP 连接关联。用户必须具有有效的电子邮件地址。

```bash
ftp_user bob@rockylinux.lan
```

* 设置 Access Control List（ACL，访问控制列表）

ACL 语法：

```bash
acl name type argument
http_access allow|deny aclname
```

示例：

```bash
acl LUNCHTIME time 12:00-14:00
http_access deny LUNCHTIME
```

在"高级配置"部分有对 ACL 的更广泛讨论。

* 缓存对象最大大小 `maximum_object_size`

`maximum_object_size` 指令语法：

```bash
maximum_object_size size
```

示例：

```bash
maximum_object_size 32 MB
```

如果对象大小大于 `maximum_object_size` 限制，则对象不会被缓存。

* 代理服务器名称 `visible_hostname`

`visible_hostname` 指令语法：

```bash
visible_hostname name
```

示例：

```bash
visible_hostname proxysquid
```

!!! Note

    提供的值可能与主机名不同。

* 为 Squid 定义缓存 `cache_ufs`

```bash
cache_ufs format path size nbFolderNiv1 nbFolderNiv2
```

可以在不同文件系统上定义多个缓存以优化访问时间。

示例：

```bash
cache_dir ufs /var/spool/squid/ 100 16 256
```

| 选项 | 描述              |
|--------|--------------------------|
| ufs    | Unix 文件系统         |
| 100    | 大小（兆）             |
| 16     | 16 个顶级文件夹     |
| 256    | 256 个次级文件夹 |

当服务首次启动时，会生成缓存目录：

```bash
sudo squid -z
sudo systemctl start squid
```

### 高级配置

#### 访问控制列表（ACL）

`http_access` 指令语法：

```bash
http_access allow|deny [!]acl_name
```

示例：

```bash
http_access allow LUNCHTIME
http_access deny !LUNCHTIME
```

`!acl_name` ACL 是 `acl_name` ACL 的反面。

`acl` 指令语法：

```bash
acl name type argument
```

ACL 的顺序是累积的。具有相同名称的多个 ACL 表示单个 ACL。

示例：

午餐时间授权：

```bash
acl LUNCHTIME time 12:00-14:00
http_access allow LUNCHTIME
```

禁止视频：

```bash
acl VIDEOS rep_mime_type video/mpeg
acl VIDEOS rep_mime_type video/avi
http_access deny VIDEOS
```

管理 IP 地址：

```bash
acl XXX src 192.168.0.0/255.255.255.0
acl XXX dst 10.10.10.1
```

FQDN 管理：

```bash
acl XXX srcdomain .rockylinux.org
acl XXX dstdomain .linux.org
```

端口管理：

```bash
acl XXX port 80 21
```

协议管理：

```bash
acl XXX proto HTTP FTP
```

#### 缓存算法

存在不同的具有不同特性的缓存算法：

* LRU - *Least Recently Used*（最近最少使用）：从 RAM 中移除最旧的对象
* LRU-THOLD：根据对象大小将对象复制到缓存
* MRU：*Most Recently Used*（最近最常使用）：删除请求最少的数据
* GDSF：*Greedy Dual Size Frequency*（贪婪双尺寸频率）：根据原始大小和访问时间删除，保留最小的
* LFUDA：*Least Frequently Used With Dynamic Aging*（带动态老化的最不频繁使用）：与 GDSF 相同，但没有尺寸的概念。适用于大文件的缓存

#### 客户端认证

Squid 依赖外部程序来管理认证。它可以基于简单的平面文件如 `htpasswd` 或基于 LDAP、SMB、PAM 或其他服务。

认证也可能是法律所必需的。记得让您的用户签署使用章程！

### 工具

#### `squidclient` 命令

使用 `squidclient` 命令测试对 Squid 服务器的请求。

`squidclient` 命令语法：

```bash
squidclient [-s] [-h target] [-p port] url
```

示例：

```bash
squidclient -s -h localhost -p 8080 http://localhost/
```

| 选项 | 描述                                   |
|--------|-----------------------------------------------|
| `-s`   | 静默模式（控制台不显示任何内容） |
| `-h`   | 定义目标代理                          |
| `-p`   | 监听端口（默认 3128）                 |
| `-r`   | 强制服务器重新加载对象        |

#### 分析日志

您可以使用以下命令监控 Squid 的日志记录：

```bash
tail -f /var/log/squid/access.log
```

日志行分解：

| 选项        | 描述                           |
|---------------|---------------------------------------|
| Date          | 日志时间戳                         |
| Response time | 请求响应时间             |
| @client       | 客户端 IP 地址                     |
| Status code   | HTTP 响应码                    |
| Size          | 传输大小                         |
| Method        | HTTP 方法（Put / Get / Post / 等） |
| URL           | 请求 URL                           |
| Peer Code     | 跨代理响应码             |
| File type     | 请求目标的 Mime 类型           |

### 安全

应为监听端口开放防火墙：

```bash
sudo firewall-cmd --add-port=3128/tcp --permanent
sudo firewall-cmd --reload
```

### 实践坊

在本实践坊中，您将在服务器上安装 Squid 并使用它下载更新。

#### 任务 1：安装和配置 Squid

安装 Squid：

```bash
sudo dnf install squid
sudo systemctl enable squid
sudo firewall-cmd --add-port=3128/tcp --permanent
sudo firewall-cmd --reload
```

在 `/etc/squid/squid.conf` 文件中删除此行注释以在磁盘上创建缓存目录：

```bash
cache_dir ufs /var/spool/squid 100 16 512
```

根据需要调整缓存大小。

创建缓存目录并启动服务。

```bash
sudo squid -z
sudo systemctl start squid
```

#### 任务 2：使用 curl 使用代理

在您的代理服务器上打开一个新终端以跟踪代理的访问。

```bash
sudo tail -f /var/log/squid/access.log
```

在第二个终端上，使用 `curl` 通过代理访问网页：

```bash
$ curl -I --proxy "http://192.168.1.10:3128" https://docs.rockylinux.org  
HTTP/1.1 200 Connection established

HTTP/2 200 
content-type: text/html
...
```

如您所见，存在两个 HTTP 连接。第一个是与代理的连接，第二个是从代理到远程服务器的连接。

您可以在第二个终端上看到痕迹：

```bash
1723793294.548     77 192.168.1.10 TCP_TUNNEL/200 3725 CONNECT docs.rockylinux.org:443 - HIER_DIRECT/151.101.122.132 -
```

这里没有缓存内容，因为您请求的是到远程服务器的 `https` 连接。

#### 任务 3：配置 DNF 使用您的代理服务器

编辑 `/etc/dnf/dnf.conf` 文件以使用 Squid 代理：

```bash
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
proxy=http://192.168.1.10:3128
```

清除您的 `dnf` 缓存并尝试更新：

```bash
sudo dnf clean all
sudo dnf update
```

在您的终端上验证 `dnf` 连接使用您的代理来下载其更新。请注意，下表中的 "URL of repository" 将被替换为实际的镜像 URL：

```bash
1723793986.725     20 192.168.1.10 TCP_MISS/200 5238 GET "URL of repository"/9.4/extras/x86_64/os/repodata/7d78a729-8e9a-4066-96d4-ab8ed8f06ee8-FILELISTS.xml.gz - HIER_DIRECT/193.106.119.144 application/x-gzip
...
1723794176.255      1 192.168.1.10 TCP_HIT/200 655447 GET "URL of repository"/9.4/AppStream/x86_64/os/repodata/1af312c9-7139-43ed-8761-90ba3cd55461-UPDATEINFO.xml.gz - HIER_NONE/- application/x-gzip
```

在此示例中，您可以看到一个 TCP_MISS（缓存中不存在）的连接和另一个 TCP_HIT（使用缓存回答客户端）的连接。

### 总结

您现在知道如何在本地网络上安装 Squid。这将使您能够集中管理到互联网的出站连接并保护您的本地网络。

### 知识自测

:heavy_check_mark: Squid 服务器默认监听的端口是什么？

* [ ] 8080  
* [ ] 1234  
* [ ] 443  
* [ ] 3128  

:heavy_check_mark: Squid 是什么？

* [ ] 一个反向代理缓存  
* [ ] 一个代理缓存  
