---
title: Apache 加固 Web 服务器
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - apache
  - web
  - security
---

# Apache 加固 Web 服务器

## 前提条件和假设

* 一台运行 Apache 的 Rocky Linux Web 服务器
* 对在命令行中执行命令、查看日志以及其他一般系统管理员职责有较高的熟练度
* 熟练使用命令行编辑器（我们的示例使用 `vi`，通常会运行 `vim` 编辑器，但你可以替换为你喜欢的编辑器）
* 假设使用 `firewalld` 作为包过滤防火墙
* 假设使用一个网关硬件防火墙，我们的可信设备位于其后方
* 假设 Web 服务器直接应用了一个公网 IP 地址（在我们的示例中，我们使用私有 IP 地址来模拟）

## 简介

无论你是为客户托管许多网站，还是为企业托管一个重要的网站，加固你的 Web 服务器都会让你更安心，代价是管理员需要多做一些前期工作。

当很多网站由你的客户上传时，其中某个人很可能会上传一个可能存在漏洞的内容管理系统（CMS）。大多数客户注重易用性，而不是安全性，结果就是更新他们自己的 CMS 变成了完全不在优先级列表中的事项。

虽然拥有庞大 IT 团队的公司可以通知客户其 CMS 中的漏洞，但对于一个小型 IT 团队来说这可能不太现实。最好的防御手段是一台加固过的 Web 服务器。

Web 服务器加固可以采取多种形式，包括此处的任何或所有工具，也可能包括此处未定义的其他工具。

你可能会使用其中的几个工具，而不是其他所有工具。为了清晰和可读性，本文档将每个工具拆分到单独的文档中。例外是本主文档中的包过滤防火墙 (`firewalld`)。

* 一个良好的基于端口的包过滤防火墙（iptables、firewalld 或硬件防火墙——我们的示例使用 `firewalld`）。请参阅本文档后面的 `firewalld` 操作步骤。
* 一个基于主机的入侵检测系统 (HIDS)，此处为 _ossec-hids_ [Apache 加固 Web 服务器 - ossec-hids](ossec-hids.md)
* 一个基于 Web 的应用程序防火墙 (WAF)，使用 `mod_security` 规则 [Apache 加固 Web 服务器 - mod_security](modsecurity.md)
* 数据库安全（此处使用 `mariadb-server`）[MariaDB 数据库服务器](../../database/database_mariadb-server.md)
* 一个安全的 FTP 或 SFTP 服务器（此处使用 `vsftpd`）[安全 FTP 服务器 - vsftpd](../../file_sharing/secure_ftp_server_vsftpd.md) 你也可以使用 [_sftp_ 和 SSH 锁定程序](../../file_sharing/sftp.md)

本操作步骤并不取代 [Apache Web 服务器多站点设置](../apache-sites-enabled.md)，它只是向其添加这些安全元素。如果你还没有阅读过，请在继续之前花些时间查看它。

## 其他注意事项

此处概述的一些工具提供了免费和付费版本。根据你的需求或支持要求，你可能需要考虑付费版本。研究现有的选项并在权衡所有选择之后做出决定是最佳策略。

为许多这些选项购买硬件设备也是可能的。如果你不愿意折腾安装和维护自己的系统，此处概述之外的选项也都可以选择。

本文档使用了一个 `firewalld` 防火墙。`firewalld` 指南有提供。一份指南可以帮助熟悉 `iptables` 的人[将他们了解的知识迁移到 `firewalld`](../../security/firewalld.md)，另一份则更[面向初学者](../../security/firewalld-beginners.md)。在开始之前，你可能需要查看其中一份操作步骤。

你需要为你的系统调优所有这些工具。要实现这一点需要仔细监控日志，以及你的客户报告的 Web 体验。此外，你还会发现需要进行持续的调优。

这些示例使用一个私有 IP 地址来模拟公网 IP，但你也可以通过硬件防火墙上的一对一 NAT 并将 Web 服务器使用私有 IP 地址连接到该硬件防火墙（而不是连接到网关路由器）来实现相同的效果。

解释这一点需要深入了解所展示的硬件防火墙，这超出了本文档的范围。

## 约定

* **IP 地址：** 此处使用私有地址块来模拟公网 IP 地址：192.168.1.0/24，并使用局域网 IP 地址块 10.0.0.0/24。这些 IP 块无法通过互联网路由，因为它们是供私有使用的，但是如果不使用分配给某个公司或组织的真实 IP 地址，就无法模拟公网 IP 块。只需记住，就我们的目的而言，192.168.1.0/24 块是"公网" IP 块，10.0.0.0/24 是"私有" IP 块。

* **硬件防火墙：** 这是一个控制从你的可信网络对你的服务器机房设备进行访问的防火墙。这与你的包过滤防火墙不同，虽然它可能是在另一台机器上运行的另一个 `firewalld` 实例。此设备允许从我们的可信设备进行 ICMP (ping) 和 SSH (安全外壳) 连接。定义此设备超出了本文档的范围。作者曾使用 [PfSense](https://www.pfsense.org/) 和 [OPNSense](https://opnsense.org/) 安装在专用硬件上作为此设备，并取得了巨大成功。此设备将分配两个 IP 地址。一个连接到互联网路由器的模拟公网 IP (192.168.1.2)，另一个连接到我们的局域网 (10.0.0.1)。
* **互联网路由器 IP：** 用 192.168.1.1/24 模拟
* **Web 服务器 IP：** 这是分配给我们的 Web 服务器的"公网" IP 地址。再次用私有 IP 地址 192.168.1.10/24 模拟

![加固 Web 服务器](images/hardened_webserver_figure1.jpeg)

该图示显示了我们的大致布局。`firewalld` 包过滤防火墙运行在 Web 服务器上。

## 安装软件包

每个单独的软件包部分都列出了所需的安装文件和配置流程。

## 配置 `firewalld`

```bash
firewall-cmd --zone=trusted --add-source=192.168.1.2 --permanent
firewall-cmd --zone=trusted --add-service=ssh --permanent
firewall-cmd --zone=public --remove-service=ssh --permanent
firewall-cmd --zone=public --add-service=dns --permanent
firewall-cmd --zone=public --add-service=http --add-service=https --permanent
firewall-cmd --zone=public --add-service=ftp --permanent
firewall-cmd --zone=public --add-port=20/tcp --permanent
firewall-cmd --zone=public --add-port=7000-7500/tcp --permanent
firewall-cmd --reload
```

以下是发生的情况：

* 将我们的可信区域设置为硬件防火墙的 IP 地址
* 从我们的可信网络、硬件防火墙后方的设备只接受 SSH（端口 22）（只有一个 IP 地址）
* 从公共区域接受 DNS（可以通过指定服务器 IP 地址或本地 DNS 服务器（如果有的话）进一步限制）
* 从任何地方接受通过端口 80 和 443 的 Web 流量。
* 接受标准 FTP（端口 20-21）以及用于在 FTP 中交换双向通信所需的被动端口（7000-7500）。这些端口可以根据你的 FTP 服务器配置任意更改为其他端口。

    !!! note

        如今，使用 SFTP 是最佳方法。你可以从[本文档](../../file_sharing/sftp.md)了解如何[安全地使用 SFTP](../../file_sharing/sftp.md)。

* 最后重新加载防火墙

## 结论

存在许多方法来加固 Apache Web 服务器使其更安全。每一种方法彼此独立运行，因此安装和选择你需要的取决于你。

每种方法都需要一些配置和调优来满足你的特定需求。由于 Web 服务持续受到恶意行为者的攻击，实施其中至少一些方法将有助于管理员安心入眠。
