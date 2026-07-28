---
title: 9 快照服务器
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 9.4
tags:
  - incus 
  - enterprise
  - incus snapshot server
---

本章根据你正在执行的任务，结合使用了特权（root）用户和非特权（incusadmin）用户。

如开头所述，Incus 快照服务器必须在所有方面尽可能镜像生产服务器。如果主服务器的硬件发生故障，你可能需要将其投入生产，并且拥有备份和快速重启生产容器的方法可以让系统管理员免受恐慌电话和短信的骚扰。这**总是**好的！

构建快照服务器的过程与生产服务器的过程完全相同。要完全模拟你的生产服务器设置，请在快照服务器上重复第 1-4 章，完成后返回此处。

如果你在这里，说明你已经完成了快照服务器的基本安装。

## 建立主服务器和快照服务器之间的关系

在继续之前，你需要做一些整理工作。首先，如果你在生产环境中运行，你可能有 DNS 服务器的访问权限来设置 IP 到名称的解析。

在你的实验环境中，你没有这种便利。也许你的场景相同。因此，你将在主服务器和快照服务器的 `/etc/hosts` 文件中添加两个服务器的 IP 地址和名称。你必须以 root（或 _sudo_）用户身份执行此操作。

在你的实验环境中，主 Incus 服务器运行在 192.168.1.106 上，快照 Incus 服务器运行在 192.168.1.141 上。通过 SSH 连接到每台服务器，并将以下内容添加到 `/etc/hosts` 文件：

```bash
192.168.1.106 incus-primary
192.168.1.141 incus-snapshot
```

接下来，你需要允许两台服务器之间的所有流量。为此，你将更改 `firewalld` 规则。首先，在 incus-primary 服务器上添加此行：

```bash
firewall-cmd zone=trusted add-source=192.168.1.141 --permanent
```

并在快照服务器上添加此规则：

```bash
firewall-cmd zone=trusted add-source=192.168.1.106 --permanent
```

然后重新加载：

```bash
firewall-cmd reload
```

接下来，作为你的非特权（incusadmin）用户，你必须在两台机器之间建立信任关系。通过在 incus-primary 上运行以下命令来完成此操作：

```bash
incus remote add incus-snapshot
```

这将显示要接受的证书。接受它，它将提示你输入密码。这是你在 Incus 初始化步骤中设置的 "trust password"。我希望你正在跟踪所有这些密码。当你输入密码时，你将收到：

```bash
Client certificate stored at server:  incus-snapshot
```

反过来建立也没有坏处。例如，也可以在 incus-snapshot 服务器上设置信任关系。如果需要，incus-snapshot 服务器可以将快照发送回 incus-primary 服务器。重复这些步骤，并用 "incus-primary" 替换 "incus-snapshot"。

### 迁移你的第一个快照

在迁移你的第一个快照之前，你必须在 incus-snapshot 上创建你在 incus-primary 上创建的所有配置文件。在本例中，是 "macvlan" 配置文件。

你需要为 incus-snapshot 创建此配置文件。如果需要，请返回[第 6 章](06-profiles.md)并在 incus-snapshot 上创建 "macvlan" 配置文件。如果你的两台服务器具有相同的父接口名称（例如 "enp3s0"），那么你可以将 "macvlan" 配置文件复制到 incus-snapshot，而无需重新创建：

```bash
incus profile copy macvlan incus-snapshot
```

在所有关系和配置文件都设置好之后，下一步是将快照从 incus-primary 发送到 incus-snapshot。如果你一直严格跟随，你可能已经删除了所有快照。创建另一个快照：

```bash
incus snapshot rockylinux-test-9 rockylinux-test-9-snap1
```

如果你对 `incus` 运行 "info" 命令，你可以在列表底部看到快照：

```bash
incus info rockylinux-test-9
```

这将在底部显示类似以下内容：

```bash
rockylinux-test-9-snap1 at 2021/05/13 16:34 UTC) (stateless)
```

尝试迁移你的快照：

```bash
incus copy rockylinux-test-9/rockylinux-test-9-snap1 incus-snapshot:rockylinux-test-9
```

此命令表示，在容器 rockylinux-test-9 内，你想将快照 rockylinux-test-9-snap1 发送到 incus-snapshot 并将其命名为 rockylinux-test-9。

稍等片刻后，复制将完成。想确认一下吗？在 incus-snapshot 服务器上执行 `incus list`。这应返回以下内容：

```bash
+-------------------+---------+------+------+-----------+-----------+
|    NAME           |  STATE  | IPV4 | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+------+------+-----------+-----------+
| rockylinux-test-9 | STOPPED |      |      | CONTAINER | 0         |
+-------------------+---------+------+------+-----------+-----------+
```

成功！尝试启动它。因为你要在 incus-snapshot 服务器上启动它，所以你需要先在 incus-primary 服务器上停止它以避免 IP 地址冲突：

```bash
incus stop rockylinux-test-9
```

并在 incus-snapshot 服务器上：

```bash
incus start rockylinux-test-9
```

假设这一切都能毫无错误地工作，请在 incus-snapshot 上停止容器，并在 incus-primary 上再次启动它。

## 将容器的 `boot.autostart` 设置为关闭

复制到 incus-snapshot 的快照在迁移时将处于关闭状态，但如果你遇到电源事件或因更新等原因需要重启快照服务器，你将遇到问题。这些容器将尝试在快照服务器上启动，从而产生潜在的 IP 地址冲突。

为了避免这种情况，你需要设置迁移的容器，使其不会在服务器重启时启动。对于你新复制的 rockylinux-test-9 容器，你将使用以下命令执行此操作：

```bash
incus config set rockylinux-test-9 boot.autostart 0
```

对 incus-snapshot 服务器上的每个快照执行此操作。命令中的 "0" 将确保 `boot.autostart` 处于关闭状态。

## 自动化快照过程

能够在需要时创建快照是极好的，有时你*确实*需要手动创建快照。你甚至可能想手动将其复制到 incus-snapshot。但对于其他所有情况，特别是对于运行在 incus-primary 服务器上的许多容器，你**绝对不想**花一个下午的时间来删除快照服务器上的快照、创建新快照并将它们发送到快照服务器。对于你的大部分操作，你将希望自动化这一过程。

你需要在 incus-primary 上安排一个过程来自动化快照创建。你将对 incus-primary 服务器上的每个容器执行此操作。完成后，它将从此自动处理。你使用以下语法执行此操作。注意时间戳的格式与 crontab 条目的相似性：

```bash
incus config set [container_name] snapshots.schedule "50 20 * * *"
```

这表示：每天在晚上 8:50 对容器名称进行快照。

要将此应用到你的 rockylinux-test-9 容器：

```bash
incus config set rockylinux-test-9 snapshots.schedule "50 20 * * *"
```

你还想将快照的名称设置为根据你的日期有意义。Incus 在任何地方都使用 UTC，所以你跟踪事物的最佳方法是将快照名称设置为具有更易理解格式的日期和时间戳：

```bash
incus config set rockylinux-test-9 snapshots.pattern "rockylinux-test-9{{ creation_date|date:'2006-01-02_15-04-05' }}"
```

很好，但你肯定不想每天都有一个新快照而不摆脱旧快照。你会用快照填满驱动器。要解决这个问题，运行以下命令：

```bash
incus config set rockylinux-test-9 snapshots.expiry 1d
```
