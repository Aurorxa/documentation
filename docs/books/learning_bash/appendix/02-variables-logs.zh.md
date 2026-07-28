---
title: 变量 - 与日志结合使用
author: Steven Spencer
contributors: Antoine Le Morvan, Ganna Zhyrnova
tested_with: 8.5
tags:
  - bash scripting
  - bash
  - variables example
---

# 使用变量 - 日志中的实际应用

## 简介

在第二课"Bash - 使用变量"中，您已经看到了一些使用变量的方法，并学到了很多关于变量可以用于什么的知识。这只是在 bash 脚本中使用变量的一个实际示例。

## 信息

当系统管理员需要处理日志文件时，有时会遇到不同的格式。假设您想从 `dnf.log`（`/var/log/dnf.log`）中获取一些信息。让我们使用 `tail /var/log/dnf.log` 快速看一下日志文件的样子：

```
2022-05-04T09:02:18-0400 DEBUG extras: using metadata from Thu 28 Apr 2022 04:25:35 PM EDT.
2022-05-04T09:02:18-0400 DEBUG repo: using cache for: powertools
2022-05-04T09:02:18-0400 DEBUG powertools: using metadata from Thu 28 Apr 2022 04:25:36 PM EDT.
2022-05-04T09:02:18-0400 DEBUG repo: using cache for: epel
2022-05-04T09:02:18-0400 DEBUG epel: using metadata from Tue 03 May 2022 11:55:16 AM EDT.
2022-05-04T09:02:18-0400 DEBUG repo: using cache for: epel-modular
2022-05-04T09:02:18-0400 DEBUG epel-modular: using metadata from Sun 17 Apr 2022 07:09:16 PM EDT.
2022-05-04T09:02:18-0400 INFO Last metadata expiration check: 3:07:06 ago on Wed 04 May 2022 05:55:12 AM EDT.
2022-05-04T09:02:18-0400 DDEBUG timer: sack setup: 512 ms
2022-05-04T09:02:18-0400 DDEBUG Cleaning up.
```

现在看一下 `messages` 日志文件 `tail /var/log/messages`：

```
May  4 08:47:19 localhost systemd[1]: Starting dnf makecache...
May  4 08:47:19 localhost dnf[108937]: Metadata cache refreshed recently.
May  4 08:47:19 localhost systemd[1]: dnf-makecache.service: Succeeded.
May  4 08:47:19 localhost systemd[1]: Started dnf makecache.
May  4 08:51:59 localhost NetworkManager[981]: <info>  [1651668719.5310] dhcp4 (eno1): state changed extended -> extended, address=192.168.1.141
May  4 08:51:59 localhost dbus-daemon[843]: [system] Activating via systemd: service name='org.freedesktop.nm_dispatcher' unit='dbus-org.freedesktop.nm-dispatcher.service' requested by ':1.10' (uid=0 pid=981 comm="/usr/sbin/NetworkManager --no-daemon " label="system_u:system_r:NetworkManager_t:s0")
May  4 08:51:59 localhost systemd[1]: Starting Network Manager Script Dispatcher Service...
May  4 08:51:59 localhost dbus-daemon[843]: [system] Successfully activated service 'org.freedesktop.nm_dispatcher'
May  4 08:51:59 localhost systemd[1]: Started Network Manager Script Dispatcher Service.
May  4 08:52:09 localhost systemd[1]: NetworkManager-dispatcher.service: Succeeded.
```

最后让我们看一下 `date` 命令的输出：

```
Wed May  4 09:47:00 EDT 2022
```

## 发现和目标

我们可以看到，两个日志文件 `dnf.log` 和 `messages` 以完全不同的方式显示日期。如果我们想在 bash 脚本中使用 `date` 从 `messages` 日志中获取信息，我们可以毫不费力地做到，但从 `dnf.log` 中获取相同的信息则需要一些技巧。假设作为系统管理员，您需要每天检查 `dnf.log`，以确保没有您不知情或可能导致问题的内容被引入系统。您希望每天按日期从 `dnf.log` 文件中获取这些信息，然后通过电子邮件发送给您。您将使用 `cron` 作业来自动化这一过程，但首先我们需要一个能够完成我们想要的功能的脚本。

## 脚本

为了实现我们想要的功能，我们将在脚本中使用一个名为 "today" 的变量，该变量将按照 `dnf.log` 中显示的日期格式来格式化日期。为了获得正确的 `date` 格式，我们使用 `+%F`，这将为我们提供所需的 yyyy-mm-dd 格式。由于我们关心的只是日期，而不是时间或其他任何信息，这就是我们从 `dnf.log` 中获取正确信息所需的全部。先试试脚本的这一部分：

```
#!/usr/bin/env bash
# script to grab dnf.log data and send it to administrator daily

today=`date +%F`
echo $today
```

这里我们使用 `echo` 命令来查看我们是否成功完成了日期格式化。当您运行脚本时，应该会得到类似以下的今天日期的输出：

```
2022-05-04
```

如果是这样，那很好，我们可以删除我们的"调试"行并继续。让我们添加另一个名为 "logfile" 的变量，将其设置为 `/var/log/dnf.log`，然后看看是否可以使用 "today" 变量对其进行 `grep`。目前，我们只让它输出到标准输出：

```
!/usr/bin/env bash
# script to grab dnf.log data and send it to administrator daily

today=`date +%F`
logfile=/var/log/dnf.log

/bin/grep $today $logfile
```

`dnf.log` 每天包含大量信息，所以我们不在这里将其输出到屏幕上，但您应该看到只包含今天数据的输出。试试这个脚本，如果成功了，我们就可以进入下一步。在检查输出之后，下一步是我们要通过管道重定向将信息发送到电子邮件。

!!! tip

    您需要安装 `mailx` 和一个邮件守护进程（如 `postfix`）来完成下一步。此外，还*可能*需要一些配置，以便从您的服务器接收电子邮件到您公司的电子邮件地址。目前不要担心这些步骤，因为您可以检查 `maillog` 来查看是否已进行尝试，然后从那里开始让从服务器到您电子邮件地址的邮件正常工作。这不是本文档要处理的内容。现在执行：

    ```
    dnf install mailx postfix
    systemctl enable --now postfix
    ```

```
#!/usr/bin/env bash
# script to grab dnf.log data and send it to administrator daily

today=`date +%F`
logfile=/var/log/dnf.log

/bin/grep $today $logfile | /bin/mail -s "DNF logfile data for $today" systemadministrator@domain.ext
```

让我们来看看这里的脚本添加了什么。我们添加了一个管道 `|` 将输出重定向到 `/bin/mail`，使用双引号中的内容设置了电子邮件的主题（`-s`），并将收件人设置为 "systemadministrator@domain.ext"。将最后一部分替换为您的电子邮件地址，然后再次尝试运行脚本。

正如前面提到的，如果不对 Postfix 邮件设置进行一些更改，您可能收不到邮件，但您应该能在 `/var/log/maillog` 中看到尝试。

## 后续步骤

接下来您需要做的是让您的服务器能够发送电子邮件。您可以查看 [Postfix for Reporting](../../../guides/email/postfix_reporting.md) 来开始这方面的工作。我们还需要自动化这个脚本使其每天运行，为此我们将使用 `cron`。这里有几个参考文档：[cron](../../../guides/automation/cron_jobs_howto.md)、[anacron](../../../guides/automation/anacron.md) 和 [cronie](../../../guides/automation/cronie.md)。有关日期格式化的更多信息，请查看 `man date` 或[此链接](https://man7.org/linux/man-pages/man1/date.1.html)。
