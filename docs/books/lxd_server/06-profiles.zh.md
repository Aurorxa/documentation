---
title: 6 配置文件
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd profiles
---

# 第 6 章：配置文件

在本章中，你需要以非特权用户身份运行命令（如果从本手册开头开始，即为 "lxdadmin"）。

安装 LXD 时会获得一个默认配置文件，此配置文件不能被删除或修改。也就是说，你可以使用默认配置文件来创建用于容器的新配置文件。

如果你检查容器列表，你会注意到每种情况下的 IP 地址都来自桥接接口。在生产环境中，你可能想使用其他方式。这可能是从 LAN 接口分配的 DHCP 地址，甚至是静态分配的 WAN 地址。

如果你为 LXD 服务器配置两个接口，并为每个接口分配 WAN 和 LAN 上的 IP，则可以根据容器需要面对的接口来分配容器的 IP 地址。

从 Rocky Linux 9.0 版本开始（实际上是任何 Red Hat Enterprise Linux 的 bug-for-bug 副本），使用配置文件静态或动态分配 IP 地址的方法不起作用。

有一些方法可以绕过这个问题，但很烦人。这似乎与 Network Manager 中影响 macvlan 的更改有关。macvlan 允许创建许多具有不同二层地址的接口。

目前，只需注意这在使用基于 RHEL 的容器镜像时存在缺点。

## 创建 macvlan 配置文件并分配它

要创建 macvlan 配置文件，使用以下命令：

```bash
lxc profile create macvlan
```

如果你在多接口机器上，并且希望根据要访问的网络来使用多个 macvlan 模板，可以使用 "lanmacvlan" 或 "wanmacvlan" 或任何其他你想要的名称来标识配置文件。在我们的配置文件创建语句中使用 "macvlan" 完全取决于你。

你想要更改 macvlan 接口，但在此之前，需要知道 LXD 服务器的父接口是什么。这将是分配了 LAN（在此例中）IP 的接口。要找到该接口，使用：

```bash
ip addr
```

查找在 192.168.1.0/24 网络中分配了 LAN IP 的接口：

```bash
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 40:16:7e:a9:94:85 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.106/24 brd 192.168.1.255 scope global dynamic noprefixroute enp3s0
       valid_lft 4040sec preferred_lft 4040sec
    inet6 fe80::a308:acfb:fcb3:878f/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

在此例中，接口是 "enp3s0"。

接下来更改配置文件：

```bash
lxc profile device add macvlan eth0 nic nictype=macvlan parent=enp3s0
```

此命令将所有必要的参数添加到 macvlan 配置文件中以供使用。

使用以下命令检查此命令创建了什么：

```bash
lxc profile show macvlan
```

将为你提供类似以下内容的输出：

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

配置文件可用于许多其他用途，但为容器分配静态 IP，或使用你自己的 DHCP 服务器是常见需求。

要将 macvlan 配置文件分配给 rockylinux-test-8，需要执行以下操作：

```bash
lxc profile assign rockylinux-test-8 default,macvlan
```

对 rockylinux-test-9 进行相同的操作：

```bash
lxc profile assign rockylinux-test-9 default,macvlan
```

这表示你想要默认配置文件，同时还要应用 macvlan 配置文件。

## Rocky Linux macvlan

在 RHEL 发行版和克隆版中，Network Manager 一直在不断变化。因此，`macvlan` 配置文件的工作方式不起作用（至少与其他发行版相比），需要一些额外的工作才能从 DHCP 或静态分配 IP 地址。

请记住，这一切都与 Rocky Linux 无关，而是与上游软件包的实现有关。

如果你想运行 Rocky Linux 容器并使用 macvlan 从 LAN 或 WAN 网络分配 IP 地址，则过程因操作系统的容器版本（8.x 或 9.x）而异。

### Rocky Linux 9.x macvlan - DHCP 修复

首先，让我们说明在分配 macvlan 配置文件后停止和重启两个容器时会发生什么。

然而，分配了配置文件并不会改变默认配置，默认配置默认使用 DHCP。

要测试这一点，请执行以下操作：

```bash
lxc restart rocky-test-8
lxc restart rocky-test-9
```

再次列出你的容器，注意 rockylinux-test-9 不再有 IP 地址了：

```bash
lxc list
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

如你所见，Rocky Linux 8.x 容器从 LAN 接口获取了 IP 地址，而 Rocky Linux 9.x 容器却没有。

要进一步演示此问题，需要在 Rocky Linux 9.0 容器上运行 `dhclient`。这将向我们展示 macvlan 配置文件实际上**已**应用：

```bash
lxc exec rockylinux-test-9 dhclient
```

再次列出容器，现在显示如下：

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

这应该在容器停止和启动时发生，但实际上没有。假设你希望每次都使用 DHCP 分配的 IP 地址，可以通过简单的 crontab 条目来修复。为此，需要获取容器的 shell 访问权限，输入：

```bash
lxc exec rockylinux-test-9 bash
```

接下来，确定 `dhclient` 的路径。由于此容器来自最小化镜像，你需要先安装 `which`：

```bash
dnf install which
```

然后运行：

```bash
which dhclient
```

将返回：

```bash
/usr/sbin/dhclient
```

接下来，修改 root 的 crontab：

```bash
crontab -e
```

添加此行：

```bash
@reboot    /usr/sbin/dhclient
```

输入的 crontab 命令使用 *vi*。要保存更改并退出，使用 ++shift+colon+"w"+"q"++。

退出容器并重启 rockylinux-test-9：

```bash
lxc restart rockylinux-test-9
```

再次列出将显示容器已分配 DHCP 地址：

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

### Rocky Linux 9.x macvlan - 静态 IP 修复

要静态分配 IP 地址，事情变得更加复杂。由于 `network-scripts` 现在在 Rocky Linux 9.x 中已弃用，唯一的方法是通过静态分配，并且由于容器使用网络的方式，你将无法使用常规的 `ip route` 语句设置路由。问题在于应用 macvlan 配置文件时分配的接口（在此例中为 eth0）无法通过 Network Manager 管理。解决方法是重启后重命名容器上的网络接口并分配静态 IP。你可以使用脚本，并（再次）在 root 的 crontab 中运行。使用 `ip` 命令来完成。

为此，你需要再次获取容器的 shell 访问权限：

```bash
lxc exec rockylinux-test-9 bash
```

接下来，在 `/usr/local/sbin` 中创建一个名为 "static" 的 bash 脚本：

```bash
vi /usr/local/sbin/static
```

此脚本的内容并不难：

```bash
#!/usr/bin/env bash

/usr/sbin/ip link set dev eth0 name net0
/usr/sbin/ip addr add 192.168.1.151/24 dev net0
/usr/sbin/ip link set dev net0 up
/usr/sbin/ip route add default via 192.168.1.1
```

我们在这里做了什么？

* 将 eth0 重命名为我们可以管理的新名称（"net0"）
* 为容器分配新的静态 IP（192.168.1.151）
* 启动新的 "net0" 接口
* 为接口添加默认路由

使脚本可执行：

```bash
chmod +x /usr/local/sbin/static
```

使用 @reboot 时间将其添加到容器的 root crontab 中：

```bash
@reboot     /usr/local/sbin/static
```

最后，退出容器并重启它：

```bash
lxc restart rockylinux-test-9
```

等待几秒钟并再次列出容器：

```bash
lxc list
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

幸运的是，在 Ubuntu 的 Network Manager 实现中，macvlan 栈**没有**损坏。部署起来要容易得多！

就像对 rockylinux-test-9 容器一样，需要将配置文件分配给容器：

```bash
lxc profile assign ubuntu-test default,macvlan
```

要了解 DHCP 是否为容器分配了地址，请停止并再次启动容器：

```bash
lxc restart ubuntu-test
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

成功！

配置静态 IP 稍有不同，但一点也不难。你需要更改与容器连接（`10-lxc.yaml`）关联的 .yaml 文件。对于这个静态 IP，你将使用 192.168.1.201：

```bash
vi /etc/netplan/10-lxc.yaml
```

将其内容更改为以下内容：

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

保存更改并退出容器。

重启容器：

```bash
lxc restart ubuntu-test
```

再次列出容器时，你将看到静态 IP：

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

成功！

在本章使用的示例中，我们特意选择了一个难以配置的容器和两个较容易的容器。镜像列表中有更多可用的 Linux 版本。如果你有喜欢的版本，请尝试安装它，分配 macvlan 模板并设置 IP。
