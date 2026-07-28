---
title: 从 iptables 迁移到 firewalld
author: Steven Spencer
contributors: wsoyinka, Antoine Le Morvan, Ezequiel Bruni, qyecst, Ganna Zhyrnova, Neel Chauhan
update: 25-Dec-2025
tags:
  - security
  - firewalld
---

# `iptables` 用户迁移到 `firewalld` 指南 - 简介

当 `firewalld` 在 CentOS 7 中成为默认的防火墙管理器时（最初在 Fedora 18 中引入），作者继续使用 `iptables`。这有两个原因。首先，当时可用的 `firewalld` 文档使用了过于简单的规则，并没有展示 `firewalld` 如何将服务器安全锁定*到 IP 级别*。其次，作者拥有超过十年的 `iptables` 使用经验，继续使用它比学习 `firewalld` 更容易。

本文档旨在解决大多数 `firewalld` 参考文献的局限性，并迫使作者使用 `firewalld` 来模拟那些更细粒度的防火墙规则。

从手册页来看："`firewalld` 提供了一个动态管理的防火墙，支持网络/防火墙区域（zones）来定义网络连接或接口的信任级别。它支持 IPv4、IPv6 防火墙设置、以太网桥接以及运行时和永久配置选项的分离。它还支持一个接口，供服务或应用程序直接添加防火墙规则。"

`firewalld` 实际上是 Rocky Linux 中 netfilter 和 nftables 内核子系统的一个前端。

本指南侧重于将 `iptables` 防火墙的规则应用到 `firewalld` 防火墙。如果您确实处于防火墙之旅的起点，[此文档](firewalld-beginners.md)可能对您更有帮助。考虑阅读两份文档，以充分利用 `firewalld`。

## 前提条件与假设

- 在本文档中，假设您是 root 用户或具有 `sudo` 提升的权限。
- 对防火墙规则有一定了解，特别是 `iptables`，或者至少您想学习一些关于 `firewalld` 的知识。
- 您能舒适地在命令行中输入命令。
- 这里的所有示例都涉及 IPv4 IP。

## 区域

要真正理解 `firewalld`，您需要理解区域的使用。区域提供了防火墙规则集的粒度。

`firewalld` 有几个内置区域：

| zone          | 示例用途                                                                                                                                                                             |
|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| drop          | 丢弃传入连接而不回复 - 仅允许传出数据包。                                                                                                                |
| block         | 使用 icmp-host-prohibited 消息（IPv4）和 icmp6-adm-prohibited（IPv6）拒绝传入连接 - 只有在此系统内发起的网络连接是可能的。      |
| public        | 用于公共区域 - 仅接受选定的传入连接。                                                                                                                   |
| external      | 仅接受选定的传入连接，用于启用了伪装的外部网络。                                                                                      |
| dmz           | 仅接受有限的选定传入连接，用于位于隔离区（DMZ）中的可公开访问计算机，对内部网络的访问受限。                              |
| work          | 用于工作区域的计算机 - 仅接受选定的传入连接。                                                                                                              |
| home          | 用于家庭区域 - 仅接受选定的传入连接                                                                                                                      |
| internal      | 用于内部网络设备访问 - 仅接受选定的传入连接。                                                                                                   |
| trusted       | 接受所有网络连接。                                                                                                                                                        |

!!! Note

    `firewall-cmd` 是管理 `firewalld` 守护进程的命令行程序。

要列出系统上现有的区域，输入：

`firewall-cmd --get-zones`

!!! Warning

    记得检查防火墙状态，如果 `firewalld-cmd` 返回错误，可以使用：

    `firewall-cmd` 命令：

    ```
    $ firewall-cmd --state
    running
    ```

    `systemctl` 命令：

    ```
    $ systemctl status firewalld
    ```

作者不喜欢这些区域名称中的大多数。drop、block、public 和 trusted 非常清晰，但有些不够好，无法实现完美的细粒度安全。以这个 `iptables` 规则部分为例：

`iptables -A INPUT -p tcp -m tcp -s 192.168.1.122 --dport 22 -j ACCEPT`

这里您只允许一个 IP 地址通过 SSH（端口 22）进入服务器。如果您决定使用内置区域，您可以为此使用 "trusted"。首先，将 IP 添加到区域，其次，将规则应用到区域：

```bash
firewall-cmd --zone=trusted --add-source=192.168.1.122 --permanent
firewall-cmd --zone trusted --add-service=ssh --permanent
```

但是，如果在此服务器上，您还有一个仅对分配给您组织的 IP 块可访问的内联网呢？您现在是否会将 "internal" 区域应用于该规则？作者更倾向于创建一个处理管理员用户 IP（那些被允许通过安全 shell 登录服务器的人）的区域。

### 添加区域

要添加一个区域，您需要使用 `firewall-cmd` 带 `--new-zone` 参数。您将添加 "admin"（管理性的）作为一个区域：

`firewall-cmd --new-zone=admin --permanent`

!!! Note

    作者在整个过程中大量使用 `--permanent` 标志。为了测试，建议先不使用 `--permanent` 标志添加规则，测试它，如果按预期工作，然后使用 `firewall-cmd --runtime-to-permanent` 将规则移入实际运行状态，然后再运行 `firewall-cmd --reload`。如果风险较低（换句话说，您不会将自己锁定在外面），您可以如此处所做的那样添加 `--permanent` 标志。

在使用此区域之前，您需要重新加载防火墙：

`firewall-cmd --reload`

!!! tip

    关于自定义区域的说明：如果您需要添加一个将是受信任区域的区域，但仅包含特定的源 IP 或接口，不包含协议或服务，并且 "trusted" 区域不合适（可能是因为您已经将其用于其他东西等）。您可以添加自定义区域来实现这一点，但必须将区域目标从 "default" 更改为 "ACCEPT"（也可以使用 REJECT 或 DROP，取决于您的目标）。这里是一个使用 LXD 机器上的桥接接口（本例中为 lxdbr0）的示例。

    首先，您添加区域并重新加载以便可以使用它：

    ```
    firewall-cmd --new-zone=bridge --permanent
    firewall-cmd --reload
    ```

    接下来，将区域目标从 "default" 更改为 "ACCEPT"（**注意，更改目标需要 "--permanent" 选项**），然后分配接口并重新加载：

    ```
    firewall-cmd --zone=bridge --set-target=ACCEPT --permanent
    firewall-cmd --zone=bridge --add-interface=lxdbr0 --permanent
    firewall-cmd --reload
    ```

    这告诉防火墙：

    1. 正在将区域的目标更改为 ACCEPT
    2. 正在将桥接接口 "lxdbr0" 添加到区域
    3. 重新加载防火墙

    所有这些都表示您接受来自该桥接接口的所有流量。

### 列出区域

在继续之前，您需要检查列出区域的过程。您会得到一个单列输出，而不是 `iptables -L` 提供的表格输出。使用命令 `firewall-cmd --zone=[zone_name] --list-all` 列出区域。以下是列出新创建的 "admin" 区域时的样子：

`firewall-cmd --zone=admin --list-all`

```bash
admin
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services:
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

您可以通过使用以下命令列出系统上的活动区域：

`firewall-cmd --get-active-zones`

!!! Note "重要：活动区域"

    区域只有在满足以下两个条件之一时才能处于活动状态：

    1. 区域被分配到一个网络接口。
    2. 区域被分配了源 IP 或网络范围。

### 从区域中移除 IP 和服务

如果您按照之前的说明将 IP 添加到 "trusted" 区域，现在必须将其移除。记住我们关于使用 `--permanent` 标志的说明？这是避免使用它、在将此规则正式生效之前进行适当测试的好时机：

`firewall-cmd --zone=trusted --remove-source=192.168.1.122`

您也想从区域中移除 SSH 服务：

`firewall-cmd --zone=trusted --remove-service ssh`

然后测试。您想要确保在执行最后两步之前有从另一个区域通过 `ssh` 进入的方式。（参见下面的**警告**！）。如果您没有做其他更改，"public" 区域仍然会允许 SSH，因为默认情况下它就在那里。

一旦满意，将运行时规则移动到永久：

`firewall-cmd --runtime-to-permanent`

然后重新加载：

`firewall-cmd --reload`

!!! Warning

    如果您正在远程服务器或 VPS 上工作，请暂且不要执行最后一条指令！*永远不要从远程服务器移除 `ssh` 服务*，除非您有其他方式访问 shell（见下文）。

     假设您因为防火墙而锁定 SSH 访问。在这种情况下，您将需要（在最坏的情况下）亲自修复服务器、联系支持人员，或者可能需要从您的控制面板重新安装操作系统（取决于服务器是物理的还是虚拟的）。

### 使用新区域 - 添加管理 IP

现在只需使用 "admin" 区域重复我们的原始步骤：

```bash
firewall-cmd --zone=admin --add-source=192.168.1.122
firewall-cmd --zone admin --add-service=ssh
```

列出区域以确保区域看起来正确并且服务已正确添加：

`firewall-cmd --zone=admin --list-all`

测试您的规则以确保其有效。要测试：

1. 从您的源 IP（上面是 192.168.1.122）以 root 或具有 sudo 能力的用户身份 SSH（*使用 root 用户，因为您将在主机上运行需要它的命令。如果使用您的 sudo 用户，连接后请记得 `sudo -s`。*）
2. 连接后，运行 `tail /var/log/secure`，您将获得类似于以下的输出：

```bash
Feb 14 22:02:34 serverhostname sshd[9805]: Accepted password for root from 192.168.1.122 port 42854 ssh2
Feb 14 22:02:34 serverhostname sshd[9805]: pam_unix(sshd:session): session opened for user root by (uid=0)
```

这表明我们 SSH 连接的源 IP 与您刚刚添加到 "admin" 区域的 IP 相同。您可以安全地将此规则移动为永久：

`firewall-cmd --runtime-to-permanent`

完成添加规则后，重新加载：

`firewall-cmd --reload`

您可能需要将其他服务添加到 "admin" 区域，但目前 SSH 是最合理的。

!!! Warning

    默认情况下，"public" 区域启用了 `ssh` 服务；这可能是一个安全漏洞。一旦您创建了管理区域，将其分配给 `ssh`，并测试完成，您可以从 public 区域中移除该服务。

如果您需要添加多个管理 IP（相当可能），只需将其添加到区域的源中。在这种情况下，您正在向 "admin" 区域添加一个 IP：

`firewall-cmd --zone=admin --add-source=192.168.1.151 --permanent`

!!! Note

    请记住，如果您正在远程服务器或 VPS 上工作，并且互联网连接并不总是使用相同的 IP，您可能希望将您的 `ssh` 服务打开给您的互联网服务提供商或地理区域使用的 IP 地址范围。同样，这是为了您不会被自己的防火墙锁定。

    许多 ISP 对专用 IP 地址收取额外费用（如果提供的话），所以这是一个真正的担忧。

    这里的示例假设您在您自己的专用网络上使用 IP 来访问同样本地的服务器。

## ICMP 规则

检查我们 `iptables` 防火墙中要模拟到 `firewalld` 的另一行 - ICMP 规则：

`iptables -A INPUT -p icmp -m icmp --icmp-type 8 -s 192.168.1.136 -j ACCEPT`

对于其中的新手，ICMP 是一种设计用于错误报告的数据传输协议。它告诉您何时存在连接到机器的问题。

实际上，您可能会将 ICMP 打开给所有本地 IP（在这个例子中是 192.168.1.0/24）。我们的 "public" 和 "admin" 区域将默认启用 ICMP，因此限制 ICMP 到那一个网络地址的第一件事是在 "public" 和 "admin" 上阻止这些请求。

再次，这是为了演示目的。您肯定希望您的管理用户能够对您的服务器使用 ICMP，而且他们可能仍然可以，因为他们是 LAN 网络 IP 的成员。

要在 "public" 区域上关闭 ICMP：

`firewall-cmd --zone=public --add-icmp-block={echo-request,echo-reply} --permanent`

在我们的 "trusted" 区域上做同样的事情：

`firewall-cmd --zone=trusted --add-icmp-block={echo-request,echo-reply} --permanent`

这里引入了一些新东西：花括号 "{}" 允许我们指定多个参数。像往常一样，在进行此类更改后，您需要重新加载：

`firewall-cmd --reload`

使用来自不允许的 IP 的 ping 进行测试将给您：

```bash
ping 192.168.1.104
PING 192.168.1.104 (192.168.1.104) 56(84) bytes of data.
From 192.168.1.104 icmp_seq=1 Packet filtered
From 192.168.1.104 icmp_seq=2 Packet filtered
From 192.168.1.104 icmp_seq=3 Packet filtered
```

## Web 服务器端口

这里是公开允许 `http` 和 `https` 的 `iptables` 脚本，这是提供 Web 页面所需的协议：

```bash
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
```

而这是您之前可能见过多次的 `firewalld` 等效命令：

```bash
firewall-cmd --zone=public --add-service=http --add-service=https --permanent
```

这没问题，但是如果您正在运行例如 Nextcloud 服务在 http/https 上，而您只希望受信任的网络可以访问它呢？这并不少见！这种事情一直在发生，而仅仅公开允许流量，而不考虑谁实际需要访问，是一个巨大的安全风险。

您实际不能使用上面使用的 "trusted" 区域信息。那是用于测试的。您必须假设您至少将 LAN IP 块添加到了 "trusted"。它看起来像这样：

`firewall-cmd --zone=trusted --add-source=192.168.1.0/24 --permanent`

将服务添加到区域：

`firewall-cmd --zone=trusted --add-service=http --add-service=https --permanent`

如果您之前将这些服务添加到了 "public" 区域，则需要将它们移除：

`firewall-cmd --zone=public --remove-service=http --remove-service=https --permanent`

重新加载：

`firewall-cmd --reload`

## FTP 端口

回到 `iptables` 脚本。您有以下处理 FTP 的规则：

```bash
iptables -A INPUT -p tcp -m tcp --dport 20-21 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 7000-7500 -j ACCEPT
```

脚本的这一部分处理标准 FTP 端口（20 和 21）以及一些额外的被动端口。像 [VSFTPD](../file_sharing/secure_ftp_server_vsftpd.md) 这样的 FTP 服务器通常需要这类规则。通常，这种规则将位于面向公众的 Web 服务器上，用于允许来自您的客户的 ftp 连接。

`firewalld` 中不存在 ftp-data 服务（端口 20）。此处列出的端口 7000 到 7500 用于被动 FTP 连接，同样，这些在 `firewalld` 中不作为服务存在。您可以切换到 SFTP，这将简化此处的端口允许规则，并且可能是推荐的方式。

这演示了将一组 `iptables` 规则转换为 `firewalld`。要解决所有这些问题，您可以执行以下操作。

首先，将 ftp 服务添加到也托管 Web 服务的区域。在这个示例中这可能是 "public"：

`firewall-cmd --zone=public --add-service=ftp --permanent`

添加 ftp-data 端口：

`firewall-cmd --zone=public --add-port=20/tcp --permanent`

添加被动连接端口：

`firewall-cmd --zone=public --add-port=7000-7500/tcp --permanent`

然后重新加载：

`firewall-cmd --reload`

## 数据库端口

如果您在处理 Web 服务器，您几乎肯定在处理数据库。您以与应用到其他服务时相同的谨慎来处理对该数据库的访问。如果不需要从全世界访问，将您的规则应用到 "public" 以外的位置。另一个考虑是，您是否需要提供访问？同样，这可能取决于您的环境。在作者之前受雇的地方，为我们的客户使用了一个托管 Web 服务器。许多客户有 Wordpress 站点，他们中没有人真正需要或请求访问 `MariaDB` 的任何前端。如果客户需要更多访问权限，我们的解决方案是为他们的 Web 服务器创建一个 LXD 容器，按照客户想要的方式构建防火墙，并让他们负责该服务器上发生的事情。不过，如果您的服务器是公共的，您可能需要提供对 `phpmyadmin` 或其他 `MariaDB` 前端的访问。在这种情况下，您需要关注数据库的密码要求，并将数据库用户设置为非默认值。对于作者来说，密码长度是[创建密码时的首要考虑因素](https://xkcd.com/936/)。

密码安全是另一份处理该问题的文档的讨论内容。假设您对数据库访问有良好的密码策略，处理数据库的防火墙中的 `iptables` 行看起来像这样：

`iptables -A INPUT -p tcp -m tcp --dport=3600 -j ACCEPT`

 在这种情况下，将服务添加到 "public" 区域进行 `firewalld` 转换：

`firewall-cmd --zone=public --add-service=mysql --permanent`

### Postgresql 注意事项

Postgresql 使用其服务端口。这里是一个 IP tables 规则示例：

`iptables -A INPUT -p tcp -m tcp --dport 5432 -s 192.168.1.0/24 -j ACCEPT`

虽然在面向公众的 Web 服务器上较不常见，但它作为内部资源可能更常见。同样的安全注意事项适用。如果您在受信任网络（我们的示例中是 192.168.1.0/24）上有服务器，您可能不想或不需要让该网络上的所有人都能访问。Postgresql 有一个访问列表可用于更细粒度的访问权限。我们的 `firewalld` 规则看起来像这样：

`firewall-cmd --zone=trusted --add-service=postgresql`

## DNS 端口

拥有私有的或公共的 DNS 服务器也意味着在编写规则来保护这些服务时要采取预防措施。如果您有一个私有 DNS 服务器，使用的 iptables 规则看起来像这样（注意大多数 DNS 服务是 UDP 而不是 TCP，但并非总是如此）：

`iptables -A INPUT -p udp -m udp -s 192.168.1.0/24 --dport 53 -j ACCEPT`

那么仅允许您的 "trusted" 区域将是正确的。您已经设置了 "trusted" 区域的源。您所需要做的只是将服务添加到区域：

`firewall-cmd --zone=trusted --add-service=dns`

对于面向公众的 DNS 服务器，您只需要将相同的服务添加到 "public" 区域：

`firewall-cmd --zone=public --add-service=dns`

## 更多关于列出规则

!!! Note

    您*可以*列出所有规则（如果您喜欢的话），通过列出 nftables 规则。它很丑陋，我不推荐，但如果您确实必须，您可以执行 `nft list ruleset`。

到目前为止没有做太多的一件事是列出规则。这是您可以按区域执行的操作。以下是您已使用的区域的示例。请注意，您也可以在将规则移到永久之前列出区域，这是一个好主意。

`firewall-cmd --list-all --zone=trusted`

在这里您可以看到上面应用了什么：

```bash
trusted (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces:
  sources: 192.168.1.0/24
  services: dns
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks: echo-reply echo-request
  rich rules:
```

这适用于任何区域。例如，到目前为止的 "public" 区域：

`firewall-cmd --list-all --zone=public`

```bash
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: cockpit dhcpv6-client ftp http https
  ports: 20/tcp 7000-7500/tcp
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks: echo-reply echo-request
  rich rules:
```

注意您已从服务中移除了 SSH 访问并阻止了 ICMP "echo-reply" 和 "echo-request"。

到目前为止，您的 "admin" 区域看起来像这样：

`firewall-cmd --list-all --zone=admin`

```bash
  admin (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 192.168.1.122 192.168.1.151
  services: ssh
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

## 已建立的相关规则

看来 `firewalld` 内部默认处理了以下 `iptables` 规则：

`iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`

## 接口

默认情况下，`firewalld` 将监听所有可用接口。在具有面向许多网络网关的许多接口的裸机服务器上，您将需要根据它所面向的网络将接口分配给区域。

我们的示例中未添加接口，因为实验室使用 LXD 进行测试。在这种情况下，您只有一个接口可以使用。假设您的 "public" 区域需要配置为使用以太网端口 enp3s0，因为该端口上有公网 IP，并假设您的 "trusted" 和 "admin" 区域在 LAN 接口上，可能是 enp3s1。

要将这些区域分配给适当的接口，您可以使用以下命令：

```bash
firewall-cmd --zone=public --change-interface=enp3s0 --permanent
firewall-cmd --zone=trusted --change-interface=enp3s1 --permanent
firewall-cmd --zone=admin --change-interface=enp3s1 --permanent
firewall-cmd --reload
```

## 常见的 firewall-cmd 命令

您已经使用了一些命令。以下是一些更常见的命令及其作用：

| 命令                                    | 结果                                                                                                    |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------|
|`firewall-cmd --list-all-zones`             | 类似于 `firewall-cmd --list-all --zone=[zone]`，只不过它列出*所有*区域及其内容。 |
|`firewall-cmd --get-default-zone`           | 显示默认区域，除非您更改它，否则是 "public"。                                           |
|`firewall-cmd --list-services --zone=[zone]`| 显示为该区域启用的所有服务。                                                           |
|`firewall-cmd --list-ports --zone=[zone]`   | 显示该区域上打开的所有端口。                                                                         |
|`firewall-cmd --get-active-zones`           | 显示系统上的活动区域、它们的活动接口、服务和端口。                       |
|`firewall-cmd --get-services`               | 显示所有可用的可能用于使用的服务。                                                            |
|`firewall-cmd --runtime-to-permanent`       | 如果您在没有 `--permanent` 选项的情况下输入了许多规则，请在重新加载前执行此操作。                |

这里没有涵盖非常多的 `firewall-cmd` 选项，但这为您提供了最常用的命令。

## 结论

由于 `firewalld` 是 Rocky Linux 推荐并包含的防火墙，了解它的工作原理是一个好主意。应用 `firewalld` 服务的文档中包含的简单规则通常没有考虑服务器的用途，并且除了公开允许服务之外不提供其他选项。这是一个存在不必要的安全漏洞的缺陷。

当您看到这些说明时，请考虑您的服务器的用途以及服务是否需要向全世界开放。如果没有，请考虑按上述说明在您的规则中应用更多粒度。

这不是一个详尽的 `firewalld` 指南，而是一个起点。
