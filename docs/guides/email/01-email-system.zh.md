---
title: 电子邮件系统概述
author: Steven Spencer, Franco Colussi
contributors: Ganna Zhyrnova, Colussi Franco
tested_with: 8.5, 8.6, 9.0
tags:
  - email
---

# 电子邮件系统 —— 概览与组件介绍

在过去几十年中，管理员始终需要操作电子邮件服务器。尽管诸如 GMail 和 Office 365 等现代方案已经非常普及，但电子邮件服务器依然是 Linux 管理员的必备技能。本文旨在提供指导，帮助您了解在企业中部署电子邮件系统所需的所有组件。

管理员构建邮件系统有两种主要方式：一是利用 [Postfix](http://www.postfix.org/) 和 [Dovecot](https://www.dovecot.org/) 构建单体或集群邮件系统；二是使用 [Kolab](https://kolab.org/) 或 [IRedMail](https://www.iredmail.org/) 等集成解决方案。这两种方法各有优劣。Rocky Linux 文档计划分别提供这两种方案的实施指南。本指南专注于从 Postfix 构建邮件系统。

构建 Linux 邮件系统可以由多种组件构成。以下部分将简要介绍每个组件。

## 邮件系统组件

### 邮件传输代理(Mail Transfer Agent, MTA)

邮件传输代理负责处理 SMTP (Simple Mail Transfer Protocol) 流量，完成不同邮件服务器之间的邮件发送和接收。Rocky Linux 有 `postfix` 和 `sendmail` 两个可用的 MTA。文档团队在文档中推荐使用 Postfix。

### 邮件投递代理(Mail Delivery Agent, MDA)

邮件投递代理负责将邮件投递到用户的邮箱。Dovecot 通常与 Postfix 一起使用，作为邮件存储和 IMAP/POP3 服务器。

### 邮件用户代理(Mail User Agent, MUA)

邮件用户代理是用户用来阅读邮件的客户端软件，可以是 Thunderbird、Evolution 等桌面客户端，或 Roundcube 等 Webmail 客户端，或移动设备上的电子邮件客户端。

### 反垃圾邮件和反病毒

商业和免费的反垃圾邮件(Anti-Spam)解决方案通过 MTA 来过滤邮件。SpamAssassin 是一个流行的开源解决方案。AmaVis 可以集成多种反垃圾邮件和反病毒引擎。

### DNS 记录

为了确保邮件的可靠投递，您需要正确配置以下 DNS 记录：

* **A 记录**：邮件服务器的主机名到 IP 地址的映射。
* **MX 记录**：指向您邮件服务器的邮件交换记录(Mail Exchange Record)。
* **PTR 记录**：反向 DNS 记录(reverse DNS record)，确保正向和反向 DNS 匹配。
* **SPF 记录**：发件人策略框架(Sender Policy Framework)，定义哪些服务器被允许代表您的域名发送邮件。
* **DKIM 记录**：域名密钥标识邮件(DomainKeys Identified Mail)，提供加密签名以确保邮件完整性。
* **DMARC 记录**：基于域名的消息认证、报告和一致性(Domain-based Message Authentication, Reporting and Conformance)，将 SPF 和 DKIM 结合以建立邮件认证策略。

## 结论

这份概览仅触及了邮件服务器管理的一小部分。构建和维护一个功能完备的邮件服务器是一个复杂的过程，需要仔细规划和实现。本文希望为您提供一个坚实的基础，帮助您更好地理解邮件系统组件。
