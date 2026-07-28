---
title: 10 自动化快照
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 9.4
tags:
  - incus
  - enterprise
  - incus automation
---

在本章的全部内容中，你需要是 root 用户或能够通过 `sudo` 成为 root。

自动化快照过程使事情变得容易得多。

## 自动化快照复制过程

在 incus-primary 上执行此过程。你需要做的第一件事是创建一个脚本，该脚本将由 cron 在 /usr/local/sbin 中运行，名为 "refresh-containers"：

```bash
sudo vi /usr/local/sbin/refreshcontainers.sh
```

该脚本非常简洁：

```bash
#!/bin/bash
# This script is for doing an lxc copy --refresh against each container, copying
# and updating them to the snapshot server.

for x in $(/var/lib/snapd/snap/bin/lxc ls -c n --format csv)
        do echo "Refreshing $x"
        /var/lib/snapd/snap/bin/lxc copy --refresh $x incus-snapshot:$x
        done

```

使其可执行：

```bash
sudo chmod +x /usr/local/sbin/refreshcontainers.sh
```

将此脚本的所有权更改为你的 incusadmin 用户和组：

```bash
sudo chown incusadmin.incusadmin /usr/local/sbin/refreshcontainers.sh
```

为 incusadmin 用户设置 crontab 以运行此脚本，在此示例中为晚上 10 点运行：

```bash
crontab -e
```

你的条目将如下所示：

```bash
00 22 * * * /usr/local/sbin/refreshcontainers.sh > /home/incusadmin/refreshlog 2>&1
```

保存更改并退出。

这将在 incusadmin 的主目录中创建一个名为 "refreshlog" 的日志，它将让你知道你的过程是否有效。非常重要！

自动化过程有时会失败。这通常发生在特定容器刷新失败时。你可以使用以下命令手动重新运行刷新（假设 rockylinux-test-9 是我们的容器）：

```bash
lxc copy --refresh rockylinux-test-9 incus-snapshot:rockylinux-test-9
```
