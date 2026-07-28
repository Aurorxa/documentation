---
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
title: 第 7 部分. 高可用性
tags:
  - clustering
  - ha
  - high availability
  - pcs
  - pacemaker
---

## Linux 下的集群

> **高可用性**（High Availability）是 IT 领域常用的一个术语，与系统架构或服务有关，指该架构或服务具有适当的可用率。~ wikipedia

此可用性是以百分比表示的绩效衡量指标，通过比率 **运行时间** / **期望的总运行时间** 获得。

| 比率    | 年度停机时间               |
| -------- | ----------------------------- |
| 90%      | 876 小时                     |
| 95%      | 438 小时                     |
| 99%      | 87 小时 36 分钟        |
| 99,9%    | 8 小时 45 分钟 36 秒 |
| 99,99%   | 52 分钟 33 秒        |
| 99,999%  | 5 分钟 15 秒         |
| 99,9999% | 31,68 秒                 |

"高可用性"（**HA**）指为确保服务尽可能最高的可用性而采取的所有措施——即每天 24 小时正确运行。

### 概述

集群是一个"计算机集群"，由两个或更多机器组成的组。

集群允许：

* 通过使用所有节点的计算能力进行分布式计算
* 高可用性：在节点故障的情况下保证服务连续性和自动服务故障转移

#### 服务类型

* Active/Passive（主/备）服务

    使用 Pacemaker 和 DRBD 安装一个包含两个 Active/Passive 节点的集群，对于许多需要高可用性系统的场景来说是一种低成本解决方案。

* N+1 服务

    通过多个节点，Pacemaker 可以通过允许多个 Active/Passive 集群组合并共享一个备份节点来降低硬件成本。

* N TO N 服务

    通过共享存储，每个节点都可能用于容错。Pacemaker 还可以运行多个服务副本来分散工作负载。

* 远程站点服务

    Pacemaker 包含简化多站点集群创建的增强功能。

#### VIP

VIP（Virtual IP Address）是分配给 Active/Passive 集群的虚拟 IP 地址。将 VIP 分配给活动集群节点。如果发生服务故障，VIP 在故障节点上被禁用，而在接管节点上被激活。这称为故障转移（failover）。

客户端始终使用 VIP 访问集群，使活动服务器故障转移对客户端透明。

#### 脑裂（Split-brain）

脑裂是集群可能遇到的主要风险。当集群中的多个节点认为其邻居不活跃时，就会出现这种情况。然后节点尝试启动冗余服务，多个节点提供相同的服务，这可能导致恼人的副作用（网络上重复的 VIP、竞争数据访问等）。

避免此问题的可能技术解决方案是：

* 将公共网络流量与集群网络流量分开
* 使用网络绑定（network bonding）

## Pacemaker (PCS)

在本章中，您将学习 Pacemaker，一种集群解决方案。

****

**目标**：您将学习如何：

:heavy_check_mark: 安装和配置 Pacemaker 集群；
:heavy_check_mark: 管理 Pacemaker 集群。

:checkered_flag: **clustering**, **ha**, **high availability**, **pcs**, **pacemaker**

**知识**：:star: :star: :star:
**复杂性**：:star: :star:

**阅读时间**：20 分钟

****

### 概述

**Pacemaker** 是管理集群资源（VIP、服务、数据）的集群软件部分。它负责启动、停止和监督集群资源。它保证高节点可用性。

Pacemaker 使用 **corosync**（默认）或 **Heartbeat** 提供的消息层。

Pacemaker 由 **5 个关键组件**组成：

* Cluster Information Base（集群信息库，**CIB**）
* Cluster Resource Management daemon（集群资源管理守护进程，**CRMd**）
* Local Resource Management daemon（本地资源管理守护进程，**LRMd**）
* Policy Engine（策略引擎，**PEngine** 或 **PE**）
* Fencing daemon（隔离守护进程，**STONITHd**）

CIB 表示集群配置和所有集群资源的当前状态。其内容在整个集群中自动同步，并被 PEngine 用于计算如何实现理想的集群状态。

然后将指令列表提供给 Designated Controller（指定控制器，DC）。Pacemaker 通过选举其中一个 CRMd 实例作为主控来集中所有集群决策。

DC 按所需顺序执行 PEngine 的指令，将其传输到本地 LRMd 或通过 Corosync 或 Heartbeat 传输到其他节点的 CRMd。

有时可能需要停止节点以保护共享数据或启用恢复。Pacemaker 为此目的提供了 STONITHd。

#### Stonith

Stonith 是 Pacemaker 的一个组件。它代表 Shoot-The-Other-Node-In-The-Head（爆头其他节点），是一种建议实践，用于确保尽快隔离故障节点（关闭或至少与共享资源断开连接），从而避免数据损坏。

一个无响应的节点并不意味着它不能再访问数据。确保节点在将数据移交给另一个节点之前不再访问数据的唯一方法是使用 STONITH，这将关闭或重启故障服务器。

如果集群服务无法关闭，STONITH 也有作用。在这种情况下，Pacemaker 使用 STONITH 强制整个节点停止。

#### 仲裁管理（Quorum）

仲裁表示运行中的最小节点数，用于验证决策，例如决定哪个备份节点应在节点出错时接管。默认情况下，Pacemaker 要求超过一半的节点在线。

当通信问题将集群分割成多个节点组时，仲裁防止资源在超过预期的节点上启动。当集群组中已知在线的超过一半节点都在其组中时，集群具有仲裁能力（active_nodes_group > active_total_nodes / 2）。

当仲裁未达到时，默认决定是关闭所有资源。

案例研究：

* 在**两节点集群**上，由于**无法**达到仲裁，必须忽略节点故障，否则整个集群将被关闭。
* 如果 5 节点集群被分成 3 个节点和 2 个节点的两组，3 节点组将拥有仲裁并继续管理资源。
* 如果 6 节点集群被分成两个 3 节点组，则没有组拥有仲裁。在这种情况下，Pacemaker 的默认行为是停止所有资源以避免数据损坏。

#### 集群通信

Pacemaker 使用 **Corosync** 或 **Heartbeat**（来自 Linux-ha 项目）进行节点间通信和集群管理。

##### Corosync

**Corosync Cluster Engine** 是集群成员之间的消息层，集成附加功能以在应用程序内实现高可用性。Corosync 源自 OpenAIS 项目。

节点以 Client/Server 模式通过 UDP 协议通信。

它可以管理超过 16 个 Active/Passive 或 Active/Active 模式的集群。

##### Heartbeat

Heartbeat 技术比 Corosync 的功能更有限。不可能创建超过两个节点的集群，其管理规则也不如其竞争对手复杂。

!!! NOTE

    如今选择 Pacemaker/Corosync 似乎更合适，因为它已成为 RedHat、Debian 和 Ubuntu 发行版的默认选择。

#### 数据管理

##### DRDB 网络 RAID

DRDB 是一种块设备类型驱动程序，支持在网络上实现 RAID 1（镜像）。

当 NAS 或 SAN 技术不可用但需要数据同步时，DRDB 可能很有用。

### 安装

要安装 Pacemaker，首先启用 `highavailability` 仓库：

```bash
sudo dnf config-manager --set-enabled highavailability
```

关于 Pacemaker 包的一些信息：

```bash
$ dnf info pacemaker
Rocky Linux 9 - High Availability                                                                                                                                     289 kB/s | 250 kB     00:00
Available Packages
Name         : pacemaker
Version      : 2.1.7
Release      : 5.el9_4
Architecture : x86_64
Size         : 465 k
Source       : pacemaker-2.1.7-5.el9_4.src.rpm
Repository   : highavailability
Summary      : Scalable High-Availability cluster resource manager
URL          : https://www.clusterlabs.org/
License      : GPL-2.0-or-later AND LGPL-2.1-or-later
Description  : Pacemaker is an advanced, scalable High-Availability cluster resource
             : manager.
             :
             : It supports more than 16 node clusters with significant capabilities
             : for managing resources and dependencies.
             :
             : It will run scripts at initialization, when machines go up or down,
             : when related resources fail and can be configured to periodically check
             : resource health.
             :
             : Available rpmbuild rebuild options:
             :   --with(out) : cibsecrets hardening nls pre_release profiling
             :                 stonithd


```

使用 `repoquery` 命令，您可以找出 Pacemaker 包的依赖关系：

```bash
$ repoquery --requires pacemaker
corosync >= 3.1.1
pacemaker-cli = 2.1.7-5.el9_4
resource-agents
systemd
...
```

因此，Pacemaker 的安装将自动安装 Corosync 和 Pacemaker 的 CLI 接口。

关于 Corosync 包的一些信息：

```bash
$ dnf info corosync
Available Packages
Name         : corosync
Version      : 3.1.8
Release      : 1.el9
Architecture : x86_64
Size         : 262 k
Source       : corosync-3.1.8-1.el9.src.rpm
Repository   : highavailability
Summary      : The Corosync Cluster Engine and Application Programming Interfaces
URL          : http://corosync.github.io/corosync/
License      : BSD
Description  : This package contains the Corosync Cluster Engine Executive, several default
             : APIs and libraries, default configuration files, and an init script.
```

现在安装所需的包：

```bash
sudo dnf install pacemaker
```

如果您有防火墙，请开放它：

```bash
sudo firewall-cmd --permanent --add-service=high-availability
sudo firewall-cmd --reload
```

!!! NOTE

    现在不要启动服务，因为它们尚未配置，不会工作。

### 集群管理

`pcs` 包提供集群管理工具。`pcs` 命令是用于管理 **Pacemaker 高可用性堆栈**的命令行接口。

集群配置可以手动完成，但 pcs 包使得管理（创建、配置和故障排除）集群更加容易！

!!! NOTE

    除 pcs 之外还有其他替代方案。

在所有节点上安装该包并激活守护进程：

```bash
sudo dnf install pcs
sudo systemctl enable pcsd --now
```

包安装创建了一个密码为空的 `hacluster` 用户。用于执行诸如同步 Corosync 配置文件或重新启动远程节点等任务。必须为此用户分配一个密码。

```text
hacluster:x:189:189:cluster user:/var/lib/pacemaker:/sbin/nologin
```

在所有节点上，为 hacluster 用户分配相同的密码：

```bash
echo "pwdhacluster" | sudo passwd --stdin hacluster
```

!!! NOTE

    请将 "pwdhacluster" 替换为更安全的密码。

从任何节点，都可以在所有节点上作为 hacluster 用户进行认证，然后在其上使用 `pcs` 命令：

```bash
$ sudo pcs host auth server1 server2
Username: hacluster
Password:
server1: Authorized
server2: Authorized
```

从进行 pcs 认证的节点，启动集群配置：

```bash
$ sudo pcs cluster setup mycluster server1 server2
No addresses specified for host 'server1', using 'server1'
No addresses specified for host 'server2', using 'server2'
Destroying cluster on hosts: 'server1', 'server2'...
server2: Successfully destroyed cluster
server1: Successfully destroyed cluster
Requesting remove 'pcsd settings' from 'server1', 'server2'
server1: successful removal of the file 'pcsd settings'
server2: successful removal of the file 'pcsd settings'
Sending 'corosync authkey', 'pacemaker authkey' to 'server1', 'server2'
server1: successful distribution of the file 'corosync authkey'
server1: successful distribution of the file 'pacemaker authkey'
server2: successful distribution of the file 'corosync authkey'
server2: successful distribution of the file 'pacemaker authkey'
Sending 'corosync.conf' to 'server1', 'server2'
server1: successful distribution of the file 'corosync.conf'
server2: successful distribution of the file 'corosync.conf'
Cluster has been successfully set up.
```

!!! NOTE

    pcs cluster setup 命令处理两节点集群的仲裁问题。因此，这样的集群将在其中一个节点发生故障时正确运行。如果您手动配置 Corosync 或使用其他集群管理 shell，则必须正确配置 Corosync。

您现在可以启动集群：

```bash
$ sudo pcs cluster start --all
server1: Starting Cluster...
server2: Starting Cluster...
```

启用集群服务在启动时运行：

```bash
sudo pcs cluster enable --all
```

检查服务状态：

```bash
$ sudo pcs status
Cluster name: mycluster

WARNINGS:
No stonith devices and stonith-enabled is not false

Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: server1 (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  * Last updated: Mon Jul  8 17:50:14 2024 on server1
  * Last change:  Mon Jul  8 17:50:00 2024 by hacluster via hacluster on server1
  * 2 nodes configured
  * 0 resource instances configured

Node List:
  * Online: [ server1 server2 ]

Full List of Resources:
  * No resources

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```

#### 添加资源

在配置资源之前，您需要处理警告消息：

```bash
WARNINGS:
No stonith devices and stonith-enabled is not false
```

在此状态下，Pacemaker 将拒绝启动您的新资源。

您有两个选择：

* 禁用 `stonith`
* 配置它

首先，您将禁用 `stonith`，直到您学会如何配置它：

```bash
sudo pcs property set stonith-enabled=false
```

!!! WARNING

    注意不要在生产环境中将 `stonith` 保持为禁用状态！

##### VIP 配置

您将在集群上创建的第一个资源是 VIP。

使用 `pcs resource standards` 命令列出可用的标准资源：

```bash
$ pcs resource standards
lsb
ocf
service
systemd
```

此 VIP 对应于客户端的 IP 地址，以便他们可以访问未来的集群服务。您必须将其分配给其中一个节点。然后，如果发生故障，集群将把此资源从一个节点切换到另一个节点，以确保服务连续性。

```bash
pcs resource create myclusterVIP ocf:heartbeat:IPaddr2 ip=192.168.1.12 cidr_netmask=24 op monitor interval=30s
```

`ocf:heartbeat:IPaddr2` 参数包含三个字段，为 Pacemaker 提供以下内容：

* 标准（此处为 `ocf`）
* 脚本命名空间（此处为 `heartbeat`）
* 资源脚本名称

结果是将一个虚拟 IP 地址添加到托管资源列表：

```bash
$ sudo pcs status
Cluster name: mycluster

...
Cluster name: mycluster
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  ...
  * 2 nodes configured
  * 1 resource instance configured

Full List of Resources:
  * myclusterVIP        (ocf:heartbeat:IPaddr2):         Started server1
...
```

在这种情况下，VIP 在 server1 上处于活动状态。可以使用 `ip` 命令进行验证：

```bash
$ ip add show dev enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:df:29:09 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.10/24 brd 192.168.1.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet 192.168.1.12/24 brd 192.168.1.255 scope global secondary enp0s3
       valid_lft forever preferred_lft forever
```

###### 切换测试

从网络上的任何位置，在 VIP 上运行 ping 命令：

```bash
ping 192.168.1.12
```

将活动节点置于待机状态：

```bash
sudo pcs node standby server1
```

检查操作期间所有 ping 是否成功（没有丢失的 `icmp_seq`）：

```bash
64 bytes from 192.168.1.12: icmp_seq=39 ttl=64 time=0.419 ms
64 bytes from 192.168.1.12: icmp_seq=40 ttl=64 time=0.043 ms
64 bytes from 192.168.1.12: icmp_seq=41 ttl=64 time=0.129 ms
64 bytes from 192.168.1.12: icmp_seq=42 ttl=64 time=0.074 ms
64 bytes from 192.168.1.12: icmp_seq=43 ttl=64 time=0.099 ms
64 bytes from 192.168.1.12: icmp_seq=44 ttl=64 time=0.044 ms
64 bytes from 192.168.1.12: icmp_seq=45 ttl=64 time=0.021 ms
64 bytes from 192.168.1.12: icmp_seq=46 ttl=64 time=0.058 ms
```

检查集群状态：

```bash
$ sudo pcs status
Cluster name: mycluster
Cluster Summary:
...
  * 2 nodes configured
  * 1 resource instance configured

Node List:
  * Node server1: standby
  * Online: [ server2 ]

Full List of Resources:
  * myclusterVIP        (ocf:heartbeat:IPaddr2):         Started server2
```

VIP 已移动到 server2。按照之前的方法使用 `ip add` 命令检查。

将 server1 返回池中：

```bash
sudo pcs node unstandby server1
```

!!! Note 
一旦 server1 被 `unstandby`，集群将恢复到正常状态，但资源不会被转移回 server1：它仍然保持在 server2 上。

##### 服务配置

您将在集群的两个节点上安装 Apache 服务。此服务仅在活动节点上启动，如果活动节点发生故障，将与 VIP 同时切换节点。

有关详细安装说明，请参阅 Apache 章节。

您必须在两个节点上安装 `httpd`：

```bash
sudo dnf install -y httpd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

!!! WARNING

    不要自己启动或激活服务。Pacemaker 将负责处理。

一个包含服务器名称的 HTML 页面将默认显示：

```bash
echo "<html><body>Node $(hostname -f)</body></html>" | sudo tee "/var/www/html/index.html"
```

Pacemaker 资源代理将使用 `/server-status` 页面（参见 Apache 章节）来确定其健康状态。您必须通过在两个服务器上创建 `/etc/httpd/conf.d/status.conf` 文件来激活它：

```bash
sudo vim /etc/httpd/conf.d/status.conf
<Location /server-status>
    SetHandler server-status
    Require local
</Location>
```

要创建资源，您将调用 "WebSite"；您将调用 OCF 资源的 Apache 脚本，位于 heartbeat 命名空间中。

```bash
sudo pcs resource create WebSite ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf statusurl="http://localhost/server-status" op monitor interval=1min
```

集群将每分钟检查 Apache 的健康状态（`op monitor interval=1min`）。

最后，为确保 Apache 服务在与 VIP 地址相同的节点上启动，您必须向集群添加一个约束：

```bash
sudo pcs constraint colocation add WebSite with myclusterVIP INFINITY
```

也可以配置 Apache 服务在 VIP 之后启动。如果 Apache 有 VHost 配置监听 VIP 地址（`Listen 192.168.1.12`），这可能很有用：

```bash
$ sudo pcs constraint order myclusterVIP then WebSite
Adding myclusterVIP WebSite (kind: Mandatory) (Options: first-action=start then-action=start)
```

###### 测试故障转移

您将执行故障转移并测试您的 Web 服务器是否仍然可用：

```bash
$ sudo pcs status
Cluster name: mycluster
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: server1 (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  ...

Node List:
  * Online: [ server1 server2 ]

Full List of Resources:
  * myclusterVIP        (ocf:heartbeat:IPaddr2):         Started server1
  * WebSite     (ocf:heartbeat:apache):  Started server1
```

您当前正在 server1 上工作。

```bash
$ curl http://192.168.1.12/
<html><body>Node server1</body></html>
```

模拟 server1 上的故障：

```bash
sudo pcs node standby server1
```

```bash
$ curl http://192.168.1.12/
<html><body>Node server2</body></html>
```

如您所见，您的 Web 服务仍在工作，但现在在 server2 上。

```bash
sudo pcs node unstandby server1
```

请注意，服务仅在 VIP 切换和服务重新启动期间中断了几秒钟。

### 集群故障排除

#### `pcs status` 命令

`pcs status` 命令提供有关集群整体状态的信息：

```bash
$ sudo pcs status
Cluster name: mycluster
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: server1 (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  * Last updated: Tue Jul  9 12:25:42 2024 on server1
  * Last change:  Tue Jul  9 12:10:55 2024 by root via root on server1
  * 2 nodes configured
  * 2 resource instances configured

Node List:
  * Online: [ server1 ]
  * OFFLINE: [ server2 ]

Full List of Resources:
  * myclusterVIP        (ocf:heartbeat:IPaddr2):         Started server1
  * WebSite     (ocf:heartbeat:apache):  Started server1

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

如您所见，其中一个服务器处于离线状态。

#### `pcs status corosync` 命令

`pcs status corosync` 命令提供有关 `corosync` 节点状态的信息：

```bash
$ sudo pcs status corosync

Membership information
----------------------
    Nodeid      Votes Name
         1          1 server1 (local)
```

一旦 server2 恢复：

```bash
$ sudo pcs status corosync

Membership information
----------------------
    Nodeid      Votes Name
         1          1 server1 (local)
         2          1 server2
```

#### `crm_mon` 命令

`crm_mon` 命令返回集群状态信息。使用 `-1` 选项可以显示集群状态一次并退出。

```bash
$ sudo crm_mon -1
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: server1 (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  * Last updated: Tue Jul  9 12:30:21 2024 on server1
  * Last change:  Tue Jul  9 12:10:55 2024 by root via root on server1
  * 2 nodes configured
  * 2 resource instances configured

Node List:
  * Online: [ server1 server2 ]

Active Resources:
  * myclusterVIP        (ocf:heartbeat:IPaddr2):         Started server1
  * WebSite     (ocf:heartbeat:apache):  Started server1
```

#### `corosync-*cfgtool*` 命令

`corosync-cfgtool` 命令检查配置是否正确，与集群的通信是否正常工作：

```bash
$ sudo corosync-cfgtool -s
Local node ID 1, transport knet
LINK ID 0 udp
        addr    = 192.168.1.10
        status:
                nodeid:          1:     localhost
                nodeid:          2:     connected
```

`corosync-cmapctl` 命令是用于访问对象数据库的工具。
例如，您可以使用它来检查集群成员节点的状态：

```bash
$ sudo corosync-cmapctl  | grep members
runtime.members.1.config_version (u64) = 0
runtime.members.1.ip (str) = r(0) ip(192.168.1.10)
runtime.members.1.join_count (u32) = 1
runtime.members.1.status (str) = joined
runtime.members.2.config_version (u64) = 0
runtime.members.2.ip (str) = r(0) ip(192.168.1.11)
runtime.members.2.join_count (u32) = 2
runtime.members.2.status (str) = joined
```

### 实践坊

对于本实践坊，您需要两台服务器，均已按前面章节的描述安装、配置和保护了 Pacemaker 服务。

您将配置一个高可用的 Apache 集群。

您的两台服务器具有以下 IP 地址：

* server1: 192.168.1.10
* server2: 192.168.1.11

如果您没有用于解析名称的服务，请按如下方式填充 `/etc/hosts` 文件：

```bash
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.1.10 server1 server1.rockylinux.lan
192.168.1.11 server2 server2.rockylinux.lan
```

您将使用 VIP 地址 `192.168.1.12`。

#### 任务 1：安装和配置

要安装 Pacemaker，请启用 `highavailability` 仓库。

在两个节点上：

```bash
sudo dnf config-manager --set-enabled highavailability
sudo dnf install pacemaker pcs
sudo firewall-cmd --permanent --add-service=high-availability
sudo firewall-cmd --reload
sudo systemctl enable pcsd --now
echo "pwdhacluster" | sudo passwd --stdin hacluster
```

在 server1 上：

```bash
$ sudo pcs host auth server1 server2
Username: hacluster
Password:
server1: Authorized
server2: Authorized
$ sudo pcs cluster setup mycluster server1 server2
$ sudo pcs cluster start --all
$ sudo pcs cluster enable --all
$ sudo pcs property set stonith-enabled=false
```

#### 任务 2：添加 VIP

您将在集群上创建的第一个资源是 VIP。

```bash
pcs resource create myclusterVIP ocf:heartbeat:IPaddr2 ip=192.168.1.12 cidr_netmask=24 op monitor interval=30s
```

检查集群状态：

```bash
$ sudo pcs status
Cluster name: mycluster
Cluster Summary:
...
  * 2 nodes configured
  * 1 resource instance configured

Node List:
  * Node server1: standby
  * Online: [ server2 ]

Full List of Resources:
  * myclusterVIP        (ocf:heartbeat:IPaddr2):         Started server2
```

#### 任务 3：安装 Apache 服务器

在两个节点上执行此安装：

```bash
$ sudo dnf install -y httpd
$ sudo firewall-cmd --permanent --add-service=http
$ sudo firewall-cmd --reload
echo "<html><body>Node $(hostname -f)</body></html>" | sudo tee "/var/www/html/index.html"
sudo vim /etc/httpd/conf.d/status.conf
<Location /server-status>
    SetHandler server-status
    Require local
</Location>
```

#### 任务 4：添加 `httpd` 资源

仅在 server1 上，将新资源添加到集群并添加所需约束：

```bash
sudo pcs resource create WebSite ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf statusurl="http://localhost/server-status" op monitor interval=1min
sudo pcs constraint colocation add WebSite with myclusterVIP INFINITY
sudo pcs constraint order myclusterVIP then WebSite
```

#### 任务 5：测试您的集群

您将执行故障转移并测试您的 Web 服务器是否仍然可用：

```bash
$ sudo pcs status
Cluster name: mycluster
Cluster Summary:
  * Stack: corosync (Pacemaker is running)
  * Current DC: server1 (version 2.1.7-5.el9_4-0f7f88312) - partition with quorum
  ...

Node List:
  * Online: [ server1 server2 ]

Full List of Resources:
  * myclusterVIP        (ocf:heartbeat:IPaddr2):         Started server1
  * WebSite     (ocf:heartbeat:apache):  Started server1
```

您当前正在 server1 上工作。

```bash
$ curl http://192.168.1.12/
<html><body>Node server1</body></html>
```

模拟 server1 上的故障：

```bash
sudo pcs node standby server1
```

```bash
$ curl http://192.168.1.12/
<html><body>Node server2</body></html>
```

如您所见，您的 Web 服务仍在工作，但现在在 server2 上。

```bash
sudo pcs node unstandby server1
```

请注意，服务仅在 VIP 切换和服务重新启动期间中断了几秒钟。

### 知识自测

:heavy_check_mark: `pcs` 命令是控制 Pacemaker 集群的唯一命令吗？

:heavy_check_mark: 哪个命令返回集群状态？

* [ ] `sudo pcs status`
* [ ] `systemctl status pcs`
* [ ] `sudo crm_mon -1`
* [ ] `sudo pacemaker -t`
