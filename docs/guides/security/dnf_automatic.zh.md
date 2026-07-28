---
title: 使用 dnf-automatic 打补丁
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - security
  - dnf
  - automation
  - updates
---

# 使用 `dnf-automatic` 为服务器打补丁

管理安全更新的安装是系统管理员的一项重要事务。提供软件更新是一条久经考验的路径，最终很少引发问题。因此，在 Rocky 服务器上每天自动下载和应用更新是合理的。

您的信息系统安全性将得到加强。`dnf-automatic` 是一个能够帮助您实现这一目标的附加工具。

!!! tip "如果您担心..."

    多年前，像这样自动应用更新会是一场灾难的配方。经常有应用更新可能导致问题的情况。这种情况仍然偶尔发生，当软件包的更新移除了服务器上正在使用的已弃用功能时，但在大多数情况下，如今这根本不是问题。如果您仍然对让 `dnf-automatic` 处理更新感到不安，可以考虑使用它来下载和/或通知您有可用更新。这样您的服务器就不会长时间未打补丁。这些功能是 `dnf-automatic-notifyonly` 和 `dnf-automatic-download`。

    有关这些功能的更多信息，请查看[官方文档](https://dnf.readthedocs.io/en/latest/automatic.html)。

## 安装

您可以从 rocky 仓库安装 `dnf-automatic`：

```bash
sudo dnf install dnf-automatic
```

## 配置

默认情况下，更新过程将在早上 6 点开始，加上一个随机的额外时间差以避免所有机器同时更新。要更改此行为，您必须覆盖与应用程序服务关联的定时器配置：

```bash
sudo systemctl edit dnf-automatic.timer

[Unit]
Description=dnf-automatic timer
# See comment in dnf-makecache.service
ConditionPathExists=!/run/ostree-booted
Wants=network-online.target

[Timer]
OnCalendar=*-*-* 6:00
RandomizedDelaySec=10m
Persistent=true

[Install]
WantedBy=timers.target
```

此配置将启动延迟减少到早上 6:00 到 6:10 之间。（现在关机的服务器将在重启后自动打补丁。）

然后激活与服务关联的定时器（而不是服务本身）：

```bash
sudo systemctl enable --now dnf-automatic.timer
```

## CentOS 7 服务器怎么办？

!!! tip

    是的，这是 Rocky Linux 文档，但如果您是系统或网络管理员，您可能还有一些 CentOS 7 机器仍在使用中。我们理解这一点，这就是我们包含本节的原因。

CentOS 7 下的过程类似，但使用：`yum-cron`。

```bash
sudo yum install yum-cron
```

这次，服务的配置在文件 `/etc/yum/yum-cron.conf` 中完成。

根据需要设置配置：

```text
[commands]
#  What kind of update to use:
# default                            = yum upgrade
# security                           = yum --security upgrade
# security-severity:Critical         = yum --sec-severity=Critical upgrade
# minimal                            = yum --bugfix update-minimal
# minimal-security                   = yum --security update-minimal
# minimal-security-severity:Critical =  --sec-severity=Critical update-minimal
update_cmd = default

# Whether a message should be emitted when updates are available,
# were downloaded, or applied.
update_messages = yes

# Whether updates should be downloaded when they are available.
download_updates = yes

# Whether updates should be applied when they are available.  Note
# that download_updates must also be yes for the update to be applied.
apply_updates = yes

# Maximum amount of time to randomly sleep, in minutes.  The program
# will sleep for a random amount of time between 0 and random_sleep
# minutes before running.  This is useful for e.g. staggering the
# times that multiple systems will access update servers.  If
# random_sleep is 0 or negative, the program will run immediately.
# 6*60 = 360
random_sleep = 30
```

配置文件中的注释不言自明。

现在您可以启用并启动服务：

```bash
sudo systemctl enable --now yum-cron
```

## 结论

软件包的自动更新易于激活，并大大增加您的信息系统的安全性。
