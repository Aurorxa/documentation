---
title: 10 自动化快照
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd automation
---

# 第 10 章：自动化快照

在本章中，你需要以 root 用户身份操作，或者能够通过 `sudo` 成为 root。

自动化快照过程让事情变得简单很多。

## 自动化快照复制过程

在 lxd-primary 上执行此过程。首先需要做的是在 /usr/local/sbin 中创建一个脚本，该脚本将由 cron 运行，名为 "refresh-containers"：

```bash
sudo vi /usr/local/sbin/refreshcontainers.sh
```

该脚本非常简单：

```bash
#!/bin/bash
# This script is for doing an lxc copy --refresh against each container, copying
# and updating them to the snapshot server.

for x in $(/var/lib/snapd/snap/bin/lxc ls -c n --format csv)
        do echo "Refreshing $x"
        /var/lib/snapd/snap/bin/lxc copy --refresh $x lxd-snapshot:$x
        done

```

使其可执行：

```bash
sudo chmod +x /usr/local/sbin/refreshcontainers.sh
```

将此脚本的所有权更改为 lxdadmin 用户和组：

```bash
sudo chown lxdadmin.lxdadmin /usr/local/sbin/refreshcontainers.sh
```

设置 lxdadmin 用户的 crontab 来运行此脚本，此处为晚上 10 点：

```bash
crontab -e
```

你的条目将如下所示：

```bash
00 22 * * * /usr/local/sbin/refreshcontainers.sh > /home/lxdadmin/refreshlog 2>&1
```

保存更改并退出。

这将在 lxdadmin 的主目录中创建一个名为 "refreshlog" 的日志，让你了解过程是否工作正常。非常重要！

自动化过程有时会失败。这通常发生在特定容器刷新失败时。你可以使用以下命令手动重新运行刷新（此处假设 rockylinux-test-9 是我们的容器）：

```bash
lxc copy --refresh rockylinux-test-9 lxd-snapshot:rockylinux-test-9
```
