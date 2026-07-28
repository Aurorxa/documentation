---
title: Postfix 报告
author: Steven Spencer
contributors: Ganna Zhyrnova, Ezequiel Bruni
---

# Postfix 处理报告

## 引言

我们的 [基本邮件服务器文档](02-basic-email-system.md) 谈到了报告。如果您没有专用报告工具，您可能没有意识到您的邮件服务器可能正在处理大量的退信、垃圾邮件或其他异常活动，直到出现问题才后知后觉。本文档提供了一些基本的解决方法，让您无需进行重大投资即可监控邮件服务器。

## 前提条件与假设

* 具备在 Rocky Linux 服务器和命令行中舒适操作的能力。
* 能够使用编辑器修改配置文件。
* 能够以 root 身份进入机器，或通过 `sudo` 获取 root 访问权限。
* 安装 `postfix`，无论规模大小，从简单的服务器到更复杂的配置。

## Postfix 内置报告

Postfix 内置了报告，能够记录其正在处理的内容。这是一个基本的起点。

如何查看报告？使用 Perl 脚本，即 `pflogsumm`。这个脚本可以输出活动摘要。要使其工作，您需要从 EPEL 安装 `postfix-perl-scripts` 软件包：

```bash
dnf install postfix-perl-scripts
```

您还需要确保安装了 `perl`、`perl-Date-Calc`、`perl-GD` 和 `perl-GDGraph`。但它们可能是由上面安装的 `postfix-perl-scripts` RPM 作为依赖项安装的。

此软件包中的 `pflogsumm` 功能非常丰富，有很多选项。请查看[选项文档](https://raw.githubusercontent.com/badgersolutions/postfix-perl-scripts/master/pflogsumm.pl)了解全部详情。

### 基础用法

`pflogsumm` 的基本用法是直接针对您的 `maillog` 文件运行。

```bash
pflogsumm /var/log/maillog 
```

这会回显大量信息到屏幕上。您可能希望将这些信息导出到文件，以便使用其他工具或编辑器进行筛选：

```bash
pflogsumm /var/log/maillog > /root/log_summary.txt
```

!!! warning "注意"

    此文件可能包含与命名相关的邮件地址信息。在采取进一步操作（如发送给第二方）之前，请先考虑这些因素。

### 示例输出

以下是指向日志文件运行命令的示例输出：

```bash
Grand Totals
------------
messages

   1249   received
   1716   delivered
     15   forwarded
      4   deferred  (3  deferrals)
     13   bounced
    576   rejected (82%)
      0   reject warnings
      0   held
      0   discarded (0%)

   3794m  bytes received
   5739m  bytes delivered
    135   senders
    101   sending hosts/domains
    306   recipients
     37   recipient hosts/domains

message deferral detail: none

message bounce detail (by relay): none

message reject detail
---------------------

      unfriendly (81)
      bad-recipient (386)
      unknown (109)

message reject warning detail: none

message held detail: none

message discarded detail: none

smtpd (total: 8620)
-------------------
    3406   connection count
    2486   lookup
    2486   reverse
    9040   post 

    8620   other

smtp (total: 1722)
------------------
     136   hosts
       2   "noplaintext"
       0   nocc
    1686   connection
    1719   status
    1684   verify
```

## 结论

Postfix 的内置报告系统无需额外安装，可以对服务器的运行情况进行全面概览。虽然它无法替代商业的特定报告解决方案，但作为起点和监控工具来说足够了。
