---
title: 6 配置文件 (Profiles)
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with:  9.4
tags:
  - incus 
  - enterprise
  - incus profiles
---

在本章的全部内容中，你需要以你的非特权用户身份运行命令（如果你从一开始就遵循了本手册，则为 "incusadmin"）。

当你安装 Incus 时，你会获得一个默认配置文件，你不能删除或修改它。你可以使用默认配置文件为你的容器创建新的配置文件。

如果你检查你的容器列表，你会注意到每种情况下的 IP 地址都来自桥接接口。在生产环境中，你可能希望使用其他方式。这可能是来自你的 LAN 接口的 DHCP 分配的地址，或者来自你的 WAN 的静态分配的地址。

如果你配置 Incus 服务器时使用两个接口，并为每个接口分配 WAN 和 LAN 上的 IP，那么你可以根据容器需要面向的接口来分配容器的 IP 地址。

截至 Rocky Linux 版本 9.4（及任何 Red Hat Enterprise Linux 的错误复刻版），使用配置文件静态或动态分配 IP 地址的方法不工作。

有变通方法，但不太理想。这似乎与 Network Manager 中影响 `macvlan` 的更改有关。`macvlan` 允许你创建许多具有不同 Layer 2 地址的接口。

请注意，在选择基于 RHEL 的容器镜像时，这会带来一些缺点。

## 创建 `macvlan` 配置文件并分配它

要创建你的 `macvlan` 配置文件，使用此命令：

```bash
incus profile create macvlan
```

如果你是在多接口机器上，并希望根据要到达的网络使用多个 `macvlan` 模板，你可以使用 "lanmacvlan" 或 "wanmacvlan" 或任何其他你试图用来标识配置文件的名称。在配置文件创建语句中使用 "macvlan" 由你决定。

你想更改 `macvlan` 接口，但在此之前，你需要知道你的 Incus 服务器的父接口是什么。此接口将具有一个 LAN 分配的 IP（在本例中）。要找到该接口，使用：

```bash
ip addr
```

查找具有 192.168.1.0/24 网络中 LAN IP 分配的接口：

```bash
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 40:16:7e:a9:94:85 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.106/24 brd 192.168.1.255 scope global dynamic noprefixroute enp3s0
       valid_lft 4040sec preferred_lft 4040sec
    inet6 fe80::a308:acfb:fcb3:878f/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

在此例中，接口是 "enp3s0"。

接下来，更改配置文件：

```bash
incus profile device add macvlan eth0 nic nictype=macvlan parent=enp3s0
```

此命令将所有必要的参数添加到 `macvlan` 配置文件中。

通过使用以下命令检查此命令创建的内容：

```bash
incus profile show macvlan
```

这将为你提供类似以下的输出：

```bash
config: {}
description: ""
devices:
  eth0:
    nictype: macvlan
    parent: enp3s0 
    type: nic
name: macvlan
used_by: []
```

配置文件可以用于许多其他用途，但为容器分配静态 IP 或使用你自己的 DHCP 服务器是常见需求。

要将 `macvlan` 配置文件分配给 rockylinux-test-8，你需要执行以下操作：

```bash
incus profile assign rockylinux-test-8 default,macvlan
```

对 rockylinux-test-9 执行相同的操作：

```bash
incus profile assign rockylinux-test-9 default,macvlan
```

这表示你想要默认配置文件，同时也应用 `macvlan` 配置文件。

## Rocky Linux `macvlan`

Network Manager 在 RHEL 发行版和克隆版中一直在不断变化。因此，`macvlan` 配置文件的工作方式不工作（至少与其他发行版相比如此），需要额外的工作来从 DHCP 或静态分配 IP 地址。

请记住，这些主要与 Rocky Linux 无关，而是与上游包的实现有关。

如果你想运行 Rocky Linux 容器并使用 `macvlan` 从你的 LAN 或 WAN 网络分配 IP 地址，过程会因容器的操作系统版本（8.x 或 9.x）而有所不同。

### Rocky Linux 9.x macvlan - DHCP 修复

首先，让我们说明在分配 `macvlan` 配置文件后停止并重新启动两个容器时会发生什么。

但是，分配了配置文件并不会更改默认配置，默认配置默认为 DHCP。

要测试这一点，请执行以下操作：

```bash
incus restart rocky-test-8
incus restart rocky-test-9
```

再次列出你的容器，注意 rockylinux-test-9 不再具有 IP 地址：

```bash
incus list
```

```bash
+-------------------+---------+----------------------+------+-----------+-----------+
|       NAME        |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-8 | RUNNING | 192.168.1.114 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-9 | RUNNING |                      |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| ubuntu-test       | RUNNING | 10.146.84.181 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
```

如你所见，你的 Rocky Linux 8.x 容器从 LAN 接口接收到了 IP 地址，而 Rocky Linux 9.x 容器没有。

为了进一步演示此问题，你需要在 Rocky Linux 9.0 容器上运行 `dhclient`。这将向我们展示 `macvlan` 配置文件*确实*已应用：

```bash
incus exec rockylinux-test-9 dhclient
```

另一个容器列表现在显示以下内容：

```bash
+-------------------+---------+----------------------+------+-----------+-----------+
|       NAME        |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-8 | RUNNING | 192.168.1.114 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-9 | RUNNING | 192.168.1.113 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| ubuntu-test       | RUNNING | 10.146.84.181 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
```

这应该通过停止和启动容器就能发生，但实际上并没有。假设你希望每次都使用 DHCP 分配的 IP 地址，你可以通过一个简单的 crontab 条目来修复此问题。为此，你需要通过输入以下命令获得容器的 shell 访问权限：

```bash
incus shell rockylinux-test-9
```

接下来，让我们确定 `dhclient` 的路径。为此，由于此容器来自最小镜像，你需要首先安装 `which`：

```bash
dnf install which
```

然后运行：

```bash
which dhclient
```

这将返回：

```bash
/usr/sbin/dhclient
```

接下来，更改 root 的 crontab：

```bash
crontab -e
```

添加此行：

```bash
@reboot    /usr/sbin/dhclient
```

输入的 crontab 命令使用 *vi*。使用 ++shift+colon+"w"+"q"++ 保存更改并退出。

退出容器并重新启动 rockylinux-test-9：

```bash
incus restart rockylinux-test-9
```

另一个列表将显示容器已分配 DHCP 地址：

```bash
+-------------------+---------+----------------------+------+-----------+-----------+
|       NAME        |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-8 | RUNNING | 192.168.1.114 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-9 | RUNNING | 192.168.1.113 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| ubuntu-test       | RUNNING | 10.146.84.181 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+

```

### Rocky Linux 9 和 10 `macvlan` - 静态 IP 修复

在静态分配 IP 地址时，事情变得更加复杂。由于 `network-scripts` 现在在 Rocky Linux 9.x 中已被弃用，唯一的方法是通过静态赋值，并且由于容器使用网络的方式，你将无法使用普通的 `ip route` 语句来设置路由。问题在于，应用 `macvlan` 配置文件时分配的接口（在此例中为 eth0）无法通过 Network Manager 进行管理。修复方法是重启后重命名容器的网络接口并分配静态 IP。你可以使用脚本并通过 root 的 crontab 来运行。使用 `ip` 命令执行此操作。除了设置 IP 地址外，你还需要配置 DNS 以进行名称解析。同样，这并不像运行 `nmtui` 修改连接那样简单，因为该连接在 Network Manager 中不存在。解决方案是创建一个包含你要使用的 DNS 服务器的文本文件。

为此，你需要再次获得容器的 shell 访问权限：

```bash
incus shell rockylinux-test-9
```

在 `/usr/local/sbin/` 中创建一个文本文件：

```bash
vi /usr/local/sbin/dns.txt
```

将以下内容添加到文件：

```text
nameserver 8.8.8.8
nameserver 8.8.4.4
```

保存文件并退出。这表明你正在使用 Google 的开放 DNS 服务器。如果你想使用不同的 DNS 服务器，请将显示的 IP 地址替换为你首选的 DNS 服务器。

接下来，你将在 `/usr/local/sbin` 中创建一个名为 "static" 的 bash 脚本：

```bash
vi /usr/local/sbin/static
```

此脚本的内容并不复杂：

```bash
#!/usr/bin/env bash

/usr/sbin/ip link set dev eth0 name net0
/usr/sbin/ip addr add 192.168.1.151/24 dev net0
/usr/sbin/ip link set dev net0 up
sleep 2
/usr/sbin/ip route add default via 192.168.1.1
/usr/bin/cat /usr/local/sbin/dns.txt > /etc/resolv.conf
```

你在这里做了什么？

* 将 eth0 重命名为一个你可以管理的新名称（"net0"）
* 为你的容器分配了新的静态 IP（192.168.1.151）
* 启动了新的 "net0" 接口
* 在添加默认路由之前，添加了 2 秒的等待时间以确保接口已激活
* 需要为你的接口添加默认路由
* 需要填充 `resolv.conf` 文件以进行 DNS 解析

通过以下命令使你的脚本可执行：

```bash
chmod +x /usr/local/sbin/static
```

使用 @reboot 时间将此脚本添加到容器的 root crontab 中：

```bash
@reboot     /usr/local/sbin/static
```

最后，退出容器并重启它：

```bash
incus restart rockylinux-test-9
```

等待几秒钟并再次列出容器：

```bash
incus list
```

你应该看到成功：

```bash
+-------------------+---------+----------------------+------+-----------+-----------+
|       NAME        |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-8 | RUNNING | 192.168.1.114 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-9 | RUNNING | 192.168.1.151 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| ubuntu-test       | RUNNING | 10.146.84.181 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
```

## Ubuntu macvlan

幸运的是，Ubuntu 的 Network Manager 实现并没有破坏 `macvlan` 堆栈，使其部署起来容易得多！

就像你的 rockylinux-test-9 容器一样，你需要将配置文件分配给你的容器：

```bash
incus profile assign ubuntu-test default,macvlan
```

要了解 DHCP 是否将地址分配给容器，请停止并再次启动容器：

```bash
incus restart ubuntu-test
```

再次列出容器：

```bash
+-------------------+---------+----------------------+------+-----------+-----------+
|       NAME        |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-8 | RUNNING | 192.168.1.114 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-9 | RUNNING | 192.168.1.151 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| ubuntu-test       | RUNNING | 192.168.1.132 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
```

成功了！

配置静态 IP 有所不同，但并不难。你必须更改与容器连接关联的 `.yaml` 文件（`10-incus.yaml`）。对于此静态 IP，你将使用 192.168.1.201：

```bash
vi /etc/netplan/10-incus.yaml
```

将其中的内容更改为以下内容：

```bash
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses: [192.168.1.201/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
```

请保存你的更改并离开容器。

重启容器：

```bash
incus restart ubuntu-test
```

当你再次列出容器时，你将看到你的静态 IP：

```bash
+-------------------+---------+----------------------+------+-----------+-----------+
|       NAME        |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-8 | RUNNING | 192.168.1.114 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| rockylinux-test-9 | RUNNING | 192.168.1.151 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+
| ubuntu-test       | RUNNING | 192.168.1.201 (eth0) |      | CONTAINER | 0         |
+-------------------+---------+----------------------+------+-----------+-----------+

```

成功了！

在本文的示例中，特意选择了一个难以配置的容器和两个较容易的容器。镜像列表中还有更多版本的 Linux。如果你有自己偏好的版本，请尝试安装它，分配 `macvlan` 模板，并设置 IP。
