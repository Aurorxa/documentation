---
title: Tor 中继
author: Neel Chauhan
contributors: Steven Spencer
tested_with: 8.7, 9.2
tags:
  - proxy
  - proxies
---

## 简介

[Tor](https://www.torproject.org/) 是一个匿名服务和软件，通过三个由志愿者运行的服务器（称为中继）来路由流量。三跳设计旨在通过抵御监视尝试来确保隐私。

## 前提条件与假设

使用此过程的最低要求如下：

- 一个公网 IPv4 地址，无论是在服务器上直接拥有，还是通过端口转发
- 一个能够 24/7 运行的系统，以便对 Tor 网络有用
- 能够以 root 用户身份运行命令或使用 `sudo` 提升权限
- 熟悉命令行编辑器。作者在此使用 `vi` 或 `vim`，但您可以替换为您喜欢的编辑器
- 能够舒适地更改 SELinux（安全增强型 Linux）和防火墙设置
- 不限流量的连接，或具有高带宽上限的连接
- 可选：一个公网 IPv6 地址用于双栈连接

## 安装 Tor

要安装 Tor，您需要首先安装 EPEL（Extra Packages for Enterprise Linux，企业 Linux 额外软件包）并运行更新：

```bash
dnf -y install epel-release && dnf -y update
```

如果您在 Rocky Linux 10 上，需要添加 RPM 仓库。作者在此使用 `vi`，但如果您更喜欢 `nano` 或其他编辑器，可以替换使用：

```bash
vi /etc/yum.repos.d/tor.repo
```

插入以下内容：

```bash
[tor]
name=Tor for Enterprise Linux $releasever - $basearch
baseurl=https://rpm.torproject.org/centos/$releasever/$basearch
enabled=1
gpgcheck=1
gpgkey=https://rpm.torproject.org/centos/public_gpg.key
cost=100
```

然后安装 Tor：

```bash
dnf -y install tor
```

## 配置 Tor

安装软件包后，您需要配置 Tor：

```bash
vi /etc/tor/torrc
```

默认的 `torrc` 文件描述性很强，但如果您只想要一个 Tor 中继，可能会很长。一个最小的中继配置类似以下内容：

```bash
Nickname TorRelay
ORPort 9001
ContactInfo you@example.com
Log notice syslog
```

### 详细说明

- `Nickname` 是您的 Tor 中继的（非唯一的）昵称。
- `ORPort` 是您的 Tor 中继监听的 TCP 端口。默认是 `9001`。
- `ContactInfo` 是您的联系信息，以防您的 Tor 中继出现问题。将此设置为您的电子邮件地址。
- `Log` 是您的 Tor 中继日志的严重性和目的地。我们记录 `notice` 级别以防止记录敏感信息，并使用 `syslog` 输出到 `systemd` 日志。

### 系统配置

如果您选择了不同于 `9001`（默认值）的 TCP/IP 端口，您需要调整 SELinux 的 `tor_port_t` 以将您的 Tor 中继端口加入白名单。操作如下：

```bash
semanage port -a -t tor_port_t -p tcp 12345
```

将 `12345` 替换为在 `ORPort` 中设置的 TCP 端口。

您还需要在防火墙中打开您的 `ORPort` 端口。操作如下：

```bash
firewall-cmd --zone=public --add-port=9001/tcp
firewall-cmd --runtime-to-permanent
```

将 `9001` 替换为在 `ORPort` 中设置的 TCP 端口。

## 限制带宽

如果您不想将所有带宽都用于 Tor，比如您的 ISP 有公平使用政策，您可以限制带宽。您可以按带宽（例如 100 兆比特）或按一段时间内的流量（例如每天 5GB）进行限制。

为此，编辑 `torrc` 文件：

```bash
vi /etc/tor/torrc
```

如果您想限制带宽，需要将以下行追加到您的 `torrc` 文件中：

```bash
RelayBandwidthRate 12500 KB
```

这将允许每秒 12500 KB 的带宽，大约是每秒 100 兆比特。

如果您希望在一段时间内传输特定量的流量，例如每天，则改为追加以下内容：

```bash
AccountingStart day 00:00
AccountingMax 20 GB
```

这些值意味着：

- 您的带宽计费周期是每天从系统时间 00:00 开始。您也可以将 `day` 更改为 `week` 或 `month`，或用其他时间替换 `00:00`。
- 在您的带宽计费周期内，您将传输 20 GB。如果您想允许中继使用更多或更少的带宽，可以增加或减少此值。

用完指定带宽后会发生什么？您的中继将阻止新的连接尝试，直到该周期结束。如果您的中继未在周期内用完指定带宽，计数器将在没有停机时间的情况下重置。

## 测试和上线

一旦您设置了 Tor 中继配置，下一步是启动 Tor 守护进程：

```bash
systemctl enable --now tor
```

在您的 systemd 日志中，您应该得到类似以下的行：

```bash
Jan 14 15:46:36 hostname tor[1142]: Jan 14 15:46:36.000 [notice] Self-testing indicates your ORPort A.B.C.D:9001 is reachable from the outside. Excellent. Publishing server descriptor.
```

这表明您的中继是可访问的。

在几个小时内，您的中继将通过输入您的昵称或公网 IP 地址列在 [Tor 中继状态](https://metrics.torproject.org/rs.html)上。

## 中继注意事项

您还可以扩展配置，使您的 Tor 中继成为出口或桥接中继。每个公网 IP 地址最多可以设置 8 个中继。EPEL 中的 Tor systemd 单元文件不是为多个实例设计的，但可以复制和修改单元文件以适应多中继设置。

出口中继是直接连接到网站的 Tor 电路的最后一跳。桥接中继是未列出的中继，帮助受到互联网审查的用户连接到 Tor。

`torrc` 文件的选项在[手册页](https://2019.www.torproject.org/docs/tor-manual.html.en)中。这里我们描述了出口和桥接中继的基本配置。

### 运行出口中继

!!! warning

    如果您计划运行出口中继，请确保您的 ISP 或托管公司对此没有问题。来自出口中继的滥用投诉非常常见，因为它是代表 Tor 用户直接连接到网站的 Tor 电路的最后一个节点。许多 ISP 和托管公司因此不允许 Tor 出口中继。

    如果您不确定您的 ISP 是否允许 Tor 出口中继，请查看服务条款或询问您的 ISP。如果您的 ISP 说不允许，请寻找其他 ISP 或托管公司，或者考虑改用中间中继或桥接中继。

如果您想运行出口中继，需要将以下内容追加到您的 `torrc` 中：

```bash
ExitRelay 1
```

但是，这将使用以下默认出口策略：

```bash
ExitPolicy reject *:25
ExitPolicy reject *:119
ExitPolicy reject *:135-139
ExitPolicy reject *:445
ExitPolicy reject *:563
ExitPolicy reject *:1214
ExitPolicy reject *:4661-4666
ExitPolicy reject *:6346-6429
ExitPolicy reject *:6699
ExitPolicy reject *:6881-6999
ExitPolicy accept *:*
```

此出口策略仅阻止了 TCP 端口的一小部分，这允许来自 BitTorrent 和 SSH 的滥用，而许多 ISP 对此感到不安。

如果您想使用[缩减出口策略](https://gitlab.torproject.org/legacy/trac/-/wikis/doc/ReducedExitPolicy)，可以在 `torrc` 中设置：

```bash
ReducedExitPolicy 1
```

您也可以使用更严格的出口策略，例如只允许 DNS、HTTP 和 HTTPS 流量。可以按如下方式设置：

```bash
ExitPolicy accept *:53
ExitPolicy accept *:80
ExitPolicy accept *:443
ExitPolicy reject *:*
```

这些值意味着：

- 我们通过 `ExitPolicy accept` 行允许到 TCP 端口 53（DNS）、80（HTTP）和 443（HTTPS）的出口流量
- 我们通过通配符 `ExitPolicy reject` 行拒绝到任何其他 TCP 端口的出口流量

如果您想要一个宽松的出口策略，只阻止 SMTP 流量，可以按如下方式设置：

```bash
ExitPolicy reject *:25
ExitPolicy reject *:465
ExitPolicy reject *:587
ExitPolicy accpet *:*
```

这些值意味着：

- 我们在 `ExitPolicy reject` 行中拒绝到标准 SMTP TCP 端口 25、465 和 587 的出口流量
- 我们在通配符 `ExitPolicy accept` 行中允许到所有其他 TCP 端口的出口流量

我们也可以允许或阻止一系列端口，如下所示：

```bash
ExitPolicy accept *:80-81
ExitPolicy reject *:993-995
```

这些值意味着：

- 我们允许到 TCP 端口 80-81 的出口流量
- 我们拒绝到 TCP 端口 993-995 的出口流量，这些端口用于 SSL 安全的 IMAP、IRC 和 POP3 变体

您还可以允许到 IPv6 地址的出口流量，前提是您的服务器具有双栈连接：

```bash
IPv6Exit 1
```

### 运行 obfs4 桥接

在世界上许多地方，直接连接到 Tor 是被阻止的，包括中国、伊朗、俄罗斯和土库曼斯坦。在这些国家，Tor 客户端使用未列出的桥接中继。

Tor 使用可插拔传输（pluggable transports）系统运行，它允许将 Tor 流量伪装成其他协议，例如不可识别的哑流量（obfs4）、WebRTC（snowflake）或指向 Microsoft 服务的 HTTPS 连接（meek）。

由于其多功能性，obfs4 是最流行的可插拔传输协议。

要设置 obfs4 桥接，由于 obfs4 不在 EPEL 仓库中，我们需要从源代码编译它。首先安装必要的软件包：

```bash
dnf install git golang policycoreutils-python-utils
```

接下来，我们将下载并解压 obfs4 源代码：

```bash
wget https://gitlab.com/yawning/obfs4/-/archive/obfs4proxy-0.0.14/obfs4-obfs4proxy-0.0.14.tar.bz2
tar jxvf obfs4-obfs4proxy-0.0.14.tar.bz2
cd obfs4-obfs4proxy-0.0.14/obfs4proxy/
```

您也可以直接从 `git clone` 获取 obfs4，但这依赖于比 AppStream 中更新的 Go 版本，因此我们不使用该方法。

然后，我们将编译并安装 obfs4：

```bash
go build
cp -a obfs4proxy /usr/local/bin/
```

安装 obfs4 后，我们将以下内容追加到我们的 `torrc` 中：

```bash
ServerTransportPlugin obfs4 exec /usr/local/bin/obfs4proxy
ServerTransportListenAddr obfs4 0.0.0.0:12345
ExtORPort auto
```

这些值意味着：

- 我们在 `ServerTransportPlugin` 行中运行一个位于 `/usr/local/bin/obfs4proxy` 的 obfs4 可插拔传输
- `ServerTransportListenAddr` 使我们的可插拔传输在端口 12345 上监听
- 我们的 `ExtORPort` 行将在一个随机选择的端口上监听 Tor 和我们的可插拔传输之间的连接。通常，此行不应更改

如果您想在另一个 TCP 端口上监听，请将 `12345` 更改为您所需的 TCP 端口。

我们还需要在 SELinux 和 `firewalld` 中允许我们选择的 TCP 端口 `12345`（或您选择的端口）：

```bash
semanage port -a -t tor_port_t -p tcp 12345
firewall-cmd --zone=public --add-port=12345/tcp
firewall-cmd --runtime-to-permanent
```

## 运行多个中继

如前所述，每个公网 IP 地址最多可以设置 8 个 Tor 中继。例如，如果我们有 5 个公网 IP 地址，我们可以在服务器上设置最多 40 个中继。

但是，我们需要为每个运行的中继创建一个自定义的 systemd 单元文件。

现在让我们在 `/usr/lib/systemd/system/torX` 添加一个辅助 systemd 单元文件：

```bash
[Unit]
Description=Anonymizing overlay network for TCP
After=syslog.target network.target nss-lookup.target
PartOf=tor-master.service
ReloadPropagatedFrom=tor-master.service

[Service]
Type=notify
NotifyAccess=all
ExecStartPre=/usr/bin/tor --runasdaemon 0 -f /etc/tor/torrcX --DataDirectory /var/lib/tor/X --DataDirectoryGroupReadable 1 --User toranon --verify-config
ExecStart=/usr/bin/tor --runasdaemon 0 -f /etc/tor/torrcX --DataDirectory /var/lib/tor/X --DataDirectoryGroupReadable 1 --User toranon
ExecReload=/bin/kill -HUP ${MAINPID}
KillSignal=SIGINT
TimeoutSec=30
Restart=on-failure
RestartSec=1
WatchdogSec=1m
LimitNOFILE=32768

# Hardening
PrivateTmp=yes
DeviceAllow=/dev/null rw
DeviceAllow=/dev/urandom r
ProtectHome=yes
ProtectSystem=full
ReadOnlyDirectories=/run
ReadOnlyDirectories=/var
ReadWriteDirectories=/run/tor
ReadWriteDirectories=/var/lib/tor
ReadWriteDirectories=/var/log/tor
CapabilityBoundingSet=CAP_SETUID CAP_SETGID CAP_NET_BIND_SERVICE CAP_DAC_READ_SEARCH
PermissionsStartOnly=yes

[Install]
WantedBy = multi-user.target
```

将 `tor`/`torrc` 后面的 `X` 后缀替换为您想要的名称。作者喜欢用数字编号以简化，但可以是任何内容。

随后，我们将在 `/etc/tor/torrcX` 中添加该实例的 `torrc` 文件。确保每个实例使用不同的端口和/或 IP 地址。

我们还需要在 SELinux 和 `firewalld` 中允许我们选择的 TCP 端口 `12345`（或 `torrcX` 中的端口）：

```bash
semanage port -a -t tor_port_t -p tcp 12345
firewall-cmd --zone=public --add-port=12345/tcp
firewall-cmd --runtime-to-permanent
```

之后，启用 `torX` systemd 单元：

```bash
systemctl enable --now torX
```

对您想运行的每个中继重复这些步骤。

## 结论

与传统的 VPN 服务不同，Tor 利用志愿者运行的中继来确保隐私和匿名性，这正是您刚刚设置的。

虽然运行 Tor 中继确实需要一个可靠的系统，对于出口中继还需要一个支持的 ISP，但增加更多中继有助于保护隐私，同时使 Tor 更快且减少故障点。
