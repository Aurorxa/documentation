---
title: 前言
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - web
  - services
---
<!-- markdownlint-disable MD025 MD007 -->

Rocky Linux 是企业 Linux 家族的一员，特别适合托管 Web 服务，例如文件服务器（FTP、sFTP）、Web 服务器（Apache、Nginx）、应用服务器（PHP、Python）、数据库服务器（MariaDB、MySQL、PostgreSQL），或更具体的服务如负载均衡、缓存、代理或反向代理（HAProxy、Varnish、Squid）。

没有电子邮件，Web 就不会存在。Web 服务通常大量使用邮件服务器（Postfix）。

有时，这些服务会非常繁忙或需要高可用性服务。在这些情况下，可以实施其他服务以保证最佳的服务性能（Heartbeat、PCS）。

本书的每个章节均可根据您的需要独立查阅，不强制按顺序阅读各章。

本书也是一系列面向 Linux 系统管理的书籍（管理员指南、学习 Bash、学习 Ansible）的一部分。在必要时，将引导您回顾上述书籍中对应章节中可能欠缺的概念。

## 受众

本书的目标受众是已经接受过系统管理命令使用培训的系统管理员（参见[我们的管理员指南书籍](../admin_guide/00-toc.md)），希望安装、配置和保护其 Web 服务。

## 如何使用本书

本书设计为一本培训手册，您可以多种方式使用它。它可作为培训师的培训辅助工具，也可作为希望获取新技能或巩固现有知识的管理员的自学辅助工具。

要实践本书中介绍的一些服务，您可能需要两个（或更多）服务器来进行理论实践。
