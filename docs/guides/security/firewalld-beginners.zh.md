---
title: firewalld 初学者指南
author: Ezequiel Bruni
contributors: Steven Spencer, Ganna Zhyrnova
---

# 面向初学者的 `firewalld`

## 简介

很久以前，我还是一个新手计算机用户，听说有防火墙*应该*是非常好的。它应该能让我决定什么能进出我的计算机，对吧？但它似乎主要阻止我的视频游戏访问互联网；我*不*开心。

当然，如果您在这里，您可能对防火墙是什么以及它做什么比我当时有更好的理解。但如果您的防火墙经验仅仅告诉 Windows Defender 您的新应用被允许使用互联网，不用担心。如本文件标题所示，本指南是为您（和其他初学者）准备的！

所以，让我们谈谈我们在这里的目的。`firewalld` 是 Rocky Linux 自带的默认防火墙应用，它被设计得非常易于使用。您需要了解一点防火墙知识，并且不害怕使用命令行。

在这里您将学到：

- `firewalld` 如何工作的最基础知识
- 如何使用 `firewalld` 限制或允许传入和传出连接
- 如何只允许来自特定 IP 地址或位置的人远程登录您的机器
- 如何管理一些 `firewalld` 特有的功能，如区域（Zones）。

请注意，这并非旨在成为一个完整或详尽的防火墙指南；因此它只涵盖基础知识。

### 关于使用命令行管理防火墙的说明

嗯...确实存在图形化的防火墙配置选项。在桌面上，可以从仓库安装 `firewall-config`，在服务器上您可以[安装 Cockpit](https://wiki.crowncloud.net/?How_to_enable_Cockpit_Server_Manager_in_Rocky_Linux_9) 来帮助您管理防火墙和许多其他内容。**但是，出于以下几个原因，我将在本教程中教您命令行方式：**

1. 如果您运行的是服务器，您无论如何都会为大多数操作使用命令行。许多 Rocky 服务器的教程和指南将为防火墙管理提供命令行说明，您应该理解这些说明，而不仅仅是复制粘贴您看到的内容。
2. 理解 `firewalld` 命令的工作原理可能有助于您更好地掌握防火墙软件的工作方式。您可以将在更深入中学习的相同原理应用到未来的图形界面使用中，更好地理解您在做什么。

## 前提条件与假设

您需要：

- 一台任何类型的 Rocky Linux 机器，本地或远程、物理或虚拟
- 访问终端的权限，以及使用它的意愿
- 您需要 root 访问权限，或者至少能够在您的用户账户上使用 `sudo`。为简单起见，我假设所有命令都以 root 身份运行
- 对 SSH 的基本理解对于管理远程机器不会有坏处。

## 基本用法

### 系统服务命令

`firewalld` 作为一个服务在您的机器上运行。它在机器启动时启动，或者说应该启动。如果由于某种原因 `firewalld` 尚未在您的机器上启用，您可以通过一个简单的命令来启用它：

```bash
systemctl enable --now firewalld
```

`--now` 标志在启用后立即启动服务，让您可以跳过 `systemctl start firewalld` 步骤。

与 Rocky Linux 上的所有服务一样，您可以通过以下方式检查防火墙是否正在运行：

```bash
systemctl status firewalld
```

要完全停止它：

```bash
systemctl stop firewalld
```

要对服务进行硬重启：

```bash
systemctl restart firewalld
```

### 基本的 `firewalld` 配置和管理命令

`firewalld` 使用 `firewall-cmd` 命令进行配置。例如，您可以通过以下方式检查 `firewalld` 的状态：

```bash
firewall-cmd --state
```

在对防火墙进行每次*永久*更改后，您需要重新加载它以查看更改。您可以通过以下方式对防火墙配置进行"软重启"：

```bash
firewall-cmd --reload
```

!!! Note

    如果您重新加载尚未设为永久的配置，它们将会消失。

您可以通过以下方式一次性查看所有配置和设置：

```bash
firewall-cmd --list-all
```

该命令将输出类似以下内容：

```bash
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp9s0
  sources:
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

### 保存您的更改

!!! Warning "警告：请认真阅读以下内容。"

    默认情况下，所有对 `firewalld` 配置的更改都是临时的。如果您重启整个 `firewalld` 服务，或者重启您的机器，除非您执行以下两件特定事项之一，否则您的防火墙更改都不会保存。

  最佳实践是逐一测试您的更改，随着进程重新加载您的防火墙配置。如果您不小心将自己锁定在某项之外，您可以重启服务（或机器），上面提到的所有这些更改都会消失。

但是一旦您有了一个可用的配置，您可以通过以下方式永久保存您的更改：

```bash
firewall-cmd --runtime-to-permanent
```

然而，如果您对自己所做的完全确定，只想添加规则并继续做事，您可以将 `--permanent` 标志添加到任何配置命令中：

```bash
firewall-cmd --permanent [the rest of your command]
```

## 管理区域

在继续之前，我需要解释区域。区域是一种允许您为不同情况定义不同规则集的功能。区域是 `firewalld` 的重要组成部分，因此了解它们的工作原理是值得的。

如果您的机器有多种方式连接到不同的网络（例如以太网和 Wi-Fi），您可以决定一个连接比另一个更可信。您可以将以太网连接设置为 "trusted" 区域（如果它只连接到您构建的本地网络），并将 Wi-Fi（可能连接到互联网）放在 "public" 区域中并设置更严格的限制。

!!! Note

    区域只有在满足以下两个条件之一时才能处于活动状态：

    1. 区域被分配到一个网络接口
    2. 区域被分配了源 IP 或网络范围（更多信息见下文）

默认区域包括以下内容（我从 [DigitalOcean 的 `firewalld` 指南](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-8)中摘取了这些解释，您也应该阅读）：

> **drop：** 最低信任级别。所有传入连接被静默丢弃而不回复，仅允许传出连接。

> **block：** 与上述类似，但不是简单地丢弃连接，而是使用 icmp-host-prohibited 或 icmp6-adm-prohibited 消息拒绝传入请求。

> **public：** 代表公共、不信任的网络。您不信任其他计算机，但可以按需允许选定的传入连接。

> **external：** 在您将防火墙用作网关的情况下用于外部网络。它配置了 NAT 伪装，以便您的内部网络保持私有但可访问。

> **internal：** 与 external 区域的另一面，用于网关的内部部分。计算机相当可信，并且提供一些额外的服务。

> **dmz：** 用于位于 DMZ（隔离区域）中的计算机（将无法访问您网络的其余部分的隔离计算机）。仅允许某些传入连接。

> **work：** 用于工作机器。信任网络中的大多数计算机。可能会允许更多一些服务。

> **home：** 家庭环境。通常意味着您信任大多数其他计算机，并且会接受更多一些服务。

> **trusted：** 信任网络中的所有机器。这是可用选项中最开放的，应谨慎使用。

好吧，其中一些解释变得复杂了，但老实说？普通初学者可以通过理解 "trusted"、"home" 和 "public" 以及何时使用哪一个来应付。

### 区域管理命令

要查看您的默认区域，运行：

```bash
firewall-cmd --get-default-zone
```

要查看哪些区域处于活动状态并在起作用，运行：

```bash
firewall-cmd --get-active-zones
```

!!! Note "注意：其中一些可能已经为您完成。"

     如果您在 VPS 上运行 Rocky Linux，可能已经为您设置了基本配置。具体来说，您应该能够通过 SSH 访问服务器，并且网络接口将已被添加到 "public" 区域。

要更改默认区域：

```bash
firewall-cmd --set-default-zone [your-zone]
```

要将网络接口添加到区域：

```bash
firewall-cmd --zone=[your-zone] --add-interface=[your-network-device]
```

要更改网络接口的区域：

```bash
firewall-cmd --zone=[your-zone] --change-interface=[your-network-device]
```

要完全从区域中移除接口：

```bash
firewall-cmd --zone=[your-zone] --remove-interface=[your-network-device]
```

要创建您自己全新的具有完全自定义规则集的区域，并检查它是否已正确添加：

```bash
firewall-cmd --new-zone=[your-new-zone]
firewall-cmd --get-zones
```

## 管理端口

对于未入门者，端口（在此上下文中）只是计算机连接以来回发送信息的虚拟端点。可以将它们想象成计算机上的物理以太网端口或 USB 端口，但是不可见的，并且您可以同时打开多达 65,535 个。

我不会这样，但您可以。

每个端口通过一个数字来标识。一些端口被保留用于特定服务。例如，如果您曾经使用过 Web 服务器来构建网站，您可能熟悉端口 80 和端口 443。这些端口允许传输网页数据。

具体来说，端口 80 允许通过 HTTP（Hypertext Transfer Protocol，超文本传输协议）传输数据，端口 443 被保留用于 HTTPS（Hypertext Transfer Protocol Secure，安全超文本传输协议）数据。

端口 22 被保留用于 SSH（Secure Shell protocol，安全外壳协议），它允许您通过命令行登录和管理其他机器（参见[我们的简短指南](ssh_public_private_keys.md)关于此主题）。一个全新的远程服务器可能只允许通过端口 22 的 SSH 连接，而不允许其他任何内容。

其他示例包括 FTP（端口 20 和 21）、SSH（端口 22）等等。您也可以设置自定义端口供您可能安装的新应用使用，这些应用还没有标准编号。

!!! Note "注意：您不应该对所有内容都使用端口。"

    对于 SSH、HTTP/S、FTP 等，实际上建议将它们作为*服务*添加到防火墙区域，而不是作为端口号。我将在下面向您展示它是如何工作的。话虽如此，您仍然需要知道如何手动打开端口。

\* 对于绝对的初学者，HTTPS 基本上（或多或少）与 HTTP 相同，但经过加密。

### 端口管理命令

对于本节，我将使用 `--zone=public`... 以及端口 9001 作为随机示例，因为它在 9,000 以上。

要查看所有打开的端口：

```bash
firewall-cmd --list-ports
```

要将端口添加到您的防火墙区域（从而将其打开以供使用），只需运行以下命令：

```bash
firewall-cmd --zone=public --add-port=9001/tcp
```

!!! Note

    关于那个 `/tcp` 部分：

    末尾的 `/tcp` 部分告诉防火墙连接将通过 TCP（Transfer Control Protocol，传输控制协议）传入，这是您用于大多数服务器和家庭相关事务的协议。

     像 UDP 这样的替代方案用于调试或本指南范围之外的其他特定类型的事务。请参阅您具体要为其打开端口的任何应用或服务的文档。

要移除端口，只需更改一个词来反向执行命令：

```bash
firewall-cmd --zone=public --remove-port=9001/tcp
```

## 管理服务

正如您可能想象的，服务是在您的计算机上运行的相当标准化的程序。`firewalld` 被设置为可以方便地用于提供对主机上运行的常见服务的访问。

这是为这些常见服务以及更多服务打开端口的首选方式：

- HTTP 和 HTTPS：用于 Web 服务器
- FTP：用于以老式方式来回移动文件
- SSH：用于控制远程机器和以新方式来回移动文件
- Samba：用于与 Windows 机器共享文件。

!!! Warning

    **切勿从远程服务器的防火墙中移除 SSH 服务！**

    记住，SSH 是您用来登录服务器的方式。除非您有其他方式访问物理服务器或其 shell（即通过主机提供的控制面板），否则移除 SSH 服务将永久锁定您。

    您将需要联系支持人员以恢复访问权限，或者完全重新安装操作系统。

## 服务管理命令

要查看您可以潜在地添加到防火墙的所有可用服务的列表，运行：

```bash
firewall-cmd --get-services
```

要查看您当前在防火墙上激活了哪些服务，使用：

```bash
firewall-cmd --list-services
```

要在您的防火墙中打开一个服务（例如在 public 区域中的 HTTP），使用：

```bash
firewall-cmd --zone=public --add-service=http
```

要从您的防火墙移除/关闭一个服务，只需再次更改一个词：

```bash
firewall-cmd --zone=public --remove-service=http
```

!!! Note "注意：您可以添加自己的服务"

    也可以对其进行大量自定义。但这将变得相当复杂。先熟悉 `firewalld`，然后再继续深入学习。

## 限制访问

假设您有一个服务器，不想使其公开。如果您想定义谁可以通过 SSH 访问它或查看一些私有网页/应用，您可以做到这一点。

有几种方法可以实现这一目标。首先，对于更严格的服务器，您可以选择一个更严格的区域，将您的网络设备分配给它，如上所述向其添加 SSH 服务，然后像这样将您自己的公网 IP 地址加入白名单：

```bash
firewall-cmd --permanent --zone=trusted --add-source=192.168.1.0 [< 在此插入您的 IP]
```

您可以通过在末尾添加更大的数字来使其成为一个 IP 地址范围：

```bash
firewall-cmd --permanent --zone=trusted --add-source=192.168.1.0/24 [< 在此插入您的 IP]
```

再次，只需将 `--add-source` 更改为 `--remove-source` 即可反转此过程。

然而，如果您在管理一个需要公开网站的远程服务器，并且仍然只想为一个 IP 地址或一个小范围打开 SSH，您有几个选项。两个示例都将唯一的网络接口分配给 public 区域。

首先，您可以在您的 public 区域使用 "rich rule"（富规则），它看起来像这样：

```bash
# firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept'
```

一旦富规则就位，*不要*立即将规则设为永久。首先，从 public 区域配置中移除 SSH 服务，并测试您的连接以确保您仍然可以通过 SSH 访问服务器。

您的配置现在看起来应该像这样：

```bash
your@server ~# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: wlp3s0
  sources:
  services: cockpit dhcpv6-client
  ports: 80/tcp 443/tcp
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
        rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept
```

其次，您可以同时使用两个不同的区域。如果您的接口绑定到 public 区域，您可以通过添加源 IP 或 IP 范围（如上所示）来激活第二个区域（例如 "trusted" 区域）。然后，将 SSH 服务添加到 trusted 区域，并将其从 public 区域移除。

完成后，输出应该看起来有点像这样：

```bash
your@server ~# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: wlp3s0
  sources:
  services: cockpit dhcpv6-client
  ports: 80/tcp 443/tcp
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
your@server ~# firewall-cmd --list-all --zone=trusted
trusted (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 192.168.0.0/24
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

如果您被锁定，重启服务器（大多数 VPS 控制面板都有此选项）并重试。

!!! Warning

    这些技术仅在您拥有静态 IP 地址时才有效。

    如果您使用的是每次调制解调器重启时都会更改 IP 地址的互联网服务提供商，则在找到解决方案之前不要使用这些规则（至少不要用于 SSH）。您将把自己锁定在服务器之外。

    要么升级您的互联网套餐/提供商，要么获取一个为您提供专用 IP 的 VPN，并且*绝不要*丢失它。

    与此同时，[安装并配置 fail2ban](https://wiki.crowncloud.net/?How_to_Install_Fail2Ban_on_RockyLinux_8)，它可以帮助减少暴力攻击。

    显然，在您控制（并且您可以在其中手动设置每台机器的 IP 地址）的本地网络上，您可以随心所欲地使用所有这些规则。

## 最终说明

这远非一个详尽的指南，您可以通过[官方 `firewalld` 文档](https://firewalld.org/documentation/)学到更多。互联网上还有方便的应用特定指南，向您展示如何为那些特定应用设置防火墙。

对于 `iptables` 的粉丝（如果您已经读到这里...），[我们有一个指南](firewalld.md)详细介绍了 `firewalld` 和 `iptables` 在操作方式上的一些差异。该指南可能会帮助您判断是想继续使用 `firewalld` 还是回到 The Old Ways^(TM)^。在这种情况下，The Old Ways^(TM)^ 确实有可取之处。

## 结论

这就是我用尽可能少的文字来解释 `firewalld` 所有基础知识的全部内容。放慢速度，仔细实验，在您确定规则有效之前不要将它们设为永久。

还有，玩得开心。一旦掌握了基础知识，实际设置一个像样、可用的防火墙只需要 5-10 分钟。
