---
title: 9 快照服务器
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd snapshot server
---

# 第 9 章：快照服务器

本章根据你正在执行的任务混合使用特权（root）用户和非特权（lxdadmin）用户。

如开头所述，LXD 的快照服务器必须在各方面尽可能与生产服务器镜像匹配。原因是如果主服务器硬件发生故障，你可能需要将其投入生产，而拥有备份和快速重启生产容器的方法，可以让那些系统管理员的紧急电话和短信降到最低。这**总是**件好事！

构建快照服务器的过程与生产服务器完全相同。要完全模拟我们的生产服务器设置，请在快照服务器上重新执行**第 1-4 章**的所有内容，完成后回到此处。

你回来了！恭喜，这一定意味着你已经成功完成了快照服务器的基本安装。

## 设置主服务器和快照服务器之间的关系

在继续之前，需要进行一些准备工作。首先，如果你在生产环境中运行，你可能可以使用 DNS 服务器来设置 IP 到名称的解析。

在实验环境中，我们没有这种便利。也许你也面临同样的情况。因此，你将在主服务器和快照服务器的 /etc/hosts 文件中添加两个服务器的 IP 地址和名称。你需要以 root（或 _sudo_）用户身份执行此操作。

在我们的实验环境中，LXD 主服务器运行在 192.168.1.106 上，LXD 快照服务器运行在 192.168.1.141 上。通过 SSH 登录到每台服务器，并将以下内容添加到 /etc/hosts 文件中：

```bash
192.168.1.106 lxd-primary
192.168.1.141 lxd-snapshot
```

接下来，你需要允许两台服务器之间的所有流量。为此，你将更改 `firewalld` 规则。首先，在 lxd-primary 服务器上，添加此行：

```bash
firewall-cmd zone=trusted add-source=192.168.1.141 --permanent
```

在快照服务器上，添加此规则：

```bash
firewall-cmd zone=trusted add-source=192.168.1.106 --permanent
```

然后重新加载：

```bash
firewall-cmd reload
```

接下来，以非特权（lxdadmin）用户身份，你需要设置两台机器之间的信任关系。在 lxd-primary 上运行以下命令：

```bash
lxc remote add lxd-snapshot
```

这将显示证书以接受。接受它，系统会提示你输入密码。这是你在进行 LXD 初始化步骤时设置的 "trust password"。希望你能安全地跟踪所有密码。输入密码后，你将收到：

```bash
Client certificate stored at server:  lxd-snapshot
```

反向设置也很有帮助。例如，在 lxd-snapshot 服务器上也设置信任关系。这样，如果需要，lxd-snapshot 服务器可以将快照发送回 lxd-primary 服务器。重复这些步骤，将 "lxd-snapshot" 替换为 "lxd-primary"。

### 迁移你的第一个快照

在迁移第一个快照之前，需要在 lxd-snapshot 上创建你在 lxd-primary 上创建的所有配置文件。在我们的例子中，这是 "macvlan" 配置文件。

你需要为 lxd-snapshot 创建此文件。如果两部分服务器具有相同的父接口名称（例如 "enp3s0"），则你可以将 "macvlan" 配置文件复制到 lxd-snapshot 而无需重新创建：

```bash
lxc profile copy macvlan lxd-snapshot
```

所有关系和配置文件设置完毕后，下一步是实际将快照从 lxd-primary 发送到 lxd-snapshot。如果你完全按照操作要求进行，可能已经删除了所有快照。创建另一个快照：

```bash
lxc snapshot rockylinux-test-9 rockylinux-test-9-snap1
```

如果你对 `lxc` 运行 "info" 命令，你可以在列表底部看到快照：

```bash
lxc info rockylinux-test-9
```

将在底部显示类似以下内容：

```bash
rockylinux-test-9-snap1 at 2021/05/13 16:34 UTC) (stateless)
```

好的，祈祷吧！让我们尝试迁移快照：

```bash
lxc copy rockylinux-test-9/rockylinux-test-9-snap1 lxd-snapshot:rockylinux-test-9
```

此命令表示，在容器 rockylinux-test-9 中，你想将快照 rockylinux-test-9-snap1 发送到 lxd-snapshot，并将其命名为 rockylinux-test-9。

短暂时间后，复制将完成。想确定吗？在 lxd-snapshot 服务器上执行 `lxc list`。应返回以下内容：

```bash
+-------------------+---------+------+------+-----------+-----------+
|    NAME           |  STATE  | IPV4 | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+------+------+-----------+-----------+
| rockylinux-test-9 | STOPPED |      |      | CONTAINER | 0         |
+-------------------+---------+------+------+-----------+-----------+
```

成功！尝试启动它。由于我们在 lxd-snapshot 服务器上启动它，你需要先在 lxd-primary 服务器上停止它以避免 IP 地址冲突：

```bash
lxc stop rockylinux-test-9
```

在 lxd-snapshot 服务器上：

```bash
lxc start rockylinux-test-9
```

假设所有这些操作都没有错误，请在 lxd-snapshot 上停止容器并在 lxd-primary 上重新启动它。

## 将容器的 boot.autostart 设置为关闭

迁移到 lxd-snapshot 的快照在迁移时将处于停止状态，但如果由于电源事件或更新等原因需要重启快照服务器，你将遇到问题。那些容器将尝试在快照服务器上启动，从而可能产生 IP 地址冲突。

要消除此问题，你需要将迁移的容器设置为在服务器重启时不启动。对于我们新复制的 rockylinux-test-9 容器，你将执行以下操作：

```bash
lxc config set rockylinux-test-9 boot.autostart 0
```

在 lxd-snapshot 服务器上对每个快照执行此操作。命令中的 "0" 将确保 `boot.autostart` 处于关闭状态。

## 自动化快照过程

你可以在需要时创建快照，这很不错，而且有时你**确实**需要手动创建快照。你甚至可能想将其手动复制到 lxd-snapshot。但对于所有其他时间，尤其是对于 lxd-primary 服务器上运行的许多容器，你最**不**想做的事情就是花一个下午在快照服务器上删除快照、创建新快照并将其发送到快照服务器。对于你的大部分操作，你需要自动化该过程。

你首先需要做的是在 lxd-primary 上安排一个过程来自动化快照创建。你将为 lxd-primary 服务器上的每个容器执行此操作。完成后，它将从此负责处理。使用以下语法进行操作。注意时间戳与 crontab 条目的相似之处：

```bash
lxc config set [container_name] snapshots.schedule "50 20 * * *"
```

这表示每天晚 8:50 对容器名称进行快照。

将其应用于我们的 rockylinux-test-9 容器：

```bash
lxc config set rockylinux-test-9 snapshots.schedule "50 20 * * *"
```

你还需要设置快照名称为有意义的日期格式。LXD 在所有地方都使用 UTC（协调世界时），所以我们跟踪事物的最佳方式是将快照名称设置为更易理解的日期和时间戳格式：

```bash
lxc config set rockylinux-test-9 snapshots.pattern "rockylinux-test-9{{ creation_date|date:'2006-01-02_15-04-05' }}"
```

非常好，但你当然不希望每天都有新的快照而不删除旧的，对吧？否则你会用快照填满整个磁盘。要解决这个问题，运行：

```bash
lxc config set rockylinux-test-9 snapshots.expiry 1d
```
