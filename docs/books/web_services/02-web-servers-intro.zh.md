---
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
title: 第 2 部分. Web 服务器介绍
---

## 介绍

### HTTP 协议

**HTTP**（**H**yper**T**ext **T**ransfer **P**rotocol，超文本传输协议）自 1990 年以来一直是互联网上使用最广泛的协议。

该协议使得浏览器（客户端）与 Web 服务器（在 UNIX 系统上称为 `httpd`）之间能够通过一个由字符串定位的文件（主要是 HTML 格式，但也包括 CSS、JS、AVI 等）进行传输，该字符串称为 **URL**（Uniform Resource Locator，统一资源定位符）。

HTTP 是一种运行在 **TCP**（**T**ransmission **C**ontrol **P**rotocol，传输控制协议）之上的"请求-响应"协议。

1. 客户端打开与服务器的 TCP 连接并发送请求。
2. 服务器分析请求并根据其配置进行响应。

HTTP 协议是"**无状态**"的：它不会在一次请求到下一次请求之间保留任何关于客户端状态的信息。动态语言如 PHP、Python 或 Java 会将客户端会话信息存储在内存中（如电子商务网站）。

当前的 HTTP 协议版本包括广泛使用的 1.1 版本，以及正在日益普及的 2 和 3 版本。

HTTP 响应是服务器发送给浏览器的行集合。它包括：

* **状态行**：指定协议版本以及使用代码和解释文本表示的请求处理状态。该行由三个以空格分隔的元素组成：
    * 使用的协议版本
    * 状态码
    * 状态码的含义

* **响应头字段**：这些可选行提供关于响应和/或服务器的附加信息。每行由一个限定头类型的名称，后跟冒号（:）和头值组成。

* **响应体**：包含所请求的文档。

以下是一个 HTTP 响应的示例：

```bash
$ curl --head --location https://docs.rockylinux.org
HTTP/2 200
accept-ranges: bytes
access-control-allow-origin: *
age: 109725
cache-control: public, max-age=0, must-revalidate
content-disposition: inline
content-type: text/html; charset=utf-8
date: Fri, 21 Jun 2024 12:05:24 GMT
etag: "cba6b533f892339d3818dc59c3a5a69a"
server: Vercel
strict-transport-security: max-age=63072000
x-vercel-cache: HIT
x-vercel-id: cdg1::pdqbh-1718971524213-4892bf82d7b2
content-length: 154696
```

!!! NOTE

    学习如何使用 `curl` 命令对将来排查服务器故障非常有帮助。

Web 服务器的角色是将 URL 转换为本地资源。访问 <https://docs.rockylinux.org/> 页面就相当于向该机器发送一个 HTTP 请求。DNS（域名系统）服务在其中起着至关重要的作用。

### URL

**URL**（**U**niform **R**esource **L**ocator，统一资源定位符）是一个 ASCII 字符串，用于指定互联网上的资源。非正式地称为网址。

URL 有三个部分：

```text
<protocol>://<host>:<port>/<path>
```

* **协议名称**：这是用于网络通信的语言，如 HTTP、HTTPS、FTP 等。使用最广泛的协议是 HTTP（超文本传输协议）及其安全版本 HTTPS，用于交换 HTML 格式的 Web 页面。

* **登录名**和**密码**：此选项允许您指定访问安全服务器的参数。不建议使用，因为密码在 URL 中是可见的（出于安全目的）。

* **主机**：这是托管所请求资源的计算机名称。注意，也可以使用服务器的 IP 地址，但这会使 URL 的可读性降低。

* **端口号**：这与一个服务关联，使服务器能够知道所请求的资源类型。HTTP 协议的默认端口是 80 端口，HTTPS 是 443 端口。因此，当协议是 HTTP 或 HTTPS 时，端口号是可选的。

* **资源路径**：此部分让服务器知道资源的位置。通常是所请求文件的位置（目录）和名称。如果地址中没有指定任何位置，则表示主机的首页。否则，它表示要显示的页面的路径。

### 端口

HTTP 请求将到达运行在主机上的服务器的 80 端口（HTTP 的默认端口）。但是，管理员可以自由选择服务器的监听端口。

HTTP 协议有安全版本可用：HTTPS 协议（443 端口）。通过 `mod_ssl` 模块实现此加密协议。

使用其他端口也是可能的，例如端口 `8080`（Java EE 应用服务器）。

## Apache 和 Nginx

Linux 最常见的两种 Web 服务器是 Apache 和 Nginx。我们将在以下章节中讨论。
