---
title: 安装 Asterisk
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5
tags:
  - asterisk
  - pbx
  - communications
---

!!! note

    本流程最后一次测试是在 Rocky Linux 8.5 版本上。由于本流程主要依赖从 Asterisk 直接源码构建以及 Rocky Linux 的 Development Tools 组，因此应该适用于所有版本。如果遇到问题，请告知我们！

# 在 Rocky Linux 上安装 Asterisk

**什么是 Asterisk？**

Asterisk 是一个用于构建通信应用程序的开源框架(opensource framework)。此外，Asterisk 能将一台普通计算机转变为通信服务器，支持 IP PBX 系统、VoIP 网关、会议服务器和其他定制解决方案。全球范围内的中小企业、大型企业、呼叫中心、运营商和政府机构都在使用它。

Asterisk 是免费且开源的，由 [Sangoma](https://www.sangoma.com/) 赞助。Sangoma 还提供底层使用 Asterisk 的商业产品，根据您的经验和预算，使用这些产品可能比自己搭建更为有益。只有您和您的组织知道这个答案。

需要注意的是，本指南要求管理员自行进行大量研究。安装通信服务器并不难，但运维一个通信服务器可能相当复杂。虽然本指南能让您的服务器启动并运行，但它并不适合直接用于生产环境。

## 前提条件

至少需要以下技能和工具来完成本指南：

- 一台运行 Rocky Linux 的机器
- 熟悉修改配置文件和通过命令行执行命令
- 了解如何使用命令行编辑器（此处使用 `vi`，可替换为您喜欢的编辑器）
- 需要 root 访问权限，最好在终端中以 root 用户身份登录
- 来自 Fedora 的 EPEL 仓库
- 能够以 root 身份登录或通过 `sudo` 执行 root 命令。本文所有命令假定用户具有 `sudo` 权限。但配置和构建过程使用 `sudo -s` 运行。
- 要获取最新版本的 Asterisk，必须使用 `curl` 或 `wget`。本指南使用 `wget`，如果偏好，可以替换为相应的 `curl` 命令。

## 更新 Rocky Linux 并安装 `wget`

```bash
sudo dnf -y update
```

这将使您的服务器保持最新状态，包含自上次更新或安装以来发布或更新的所有软件包。然后运行：

```bash
sudo dnf install wget
```

## 设置主机名(hostname)

将主机名设置为 Asterisk 将使用的域名。

```bash
sudo hostnamectl set-hostname asterisk.example.com
```

## 添加所需仓库

首先，安装 EPEL (Extra Packages for Enterprise Linux)：

```bash
sudo dnf -y install epel-release
```

接下来，启用 Rocky Linux 的 PowerTools：

```bash
sudo dnf config-manager --set-enabled powertools
```

## 安装开发工具

```bash
sudo dnf group -y install "Development Tools"
sudo dnf -y install git wget  
```

## 安装 Asterisk

### 下载和配置 Asterisk 构建

在下载此脚本之前，请确保获取最新版本。为此，请导航到 [Asterisk 下载链接](http://downloads.asterisk.org/pub/telephony/asterisk/) 并查找最新的 Asterisk 版本。然后复制链接地址。在本文档编写时，以下是最新版本：

```bash
wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz 
tar xvfz asterisk-20-current.tar.gz
cd asterisk-20.0.0/
```

在运行下面的 `install_prereq`（以及其余命令）之前，需要成为超级用户或 root。此时更简单的做法是暂时保持 `sudo` 状态。稍后会在流程中退出 `sudo`：

```bash
sudo -s
contrib/scripts/install_prereq install
```

脚本完成后应该看到以下内容：

```text
#############################################
## install completed successfully
#############################################
```

现在已安装所有必需的软件包，下一步是配置和构建 Asterisk：

```bash
./configure --libdir=/usr/lib64 --with-jansson-bundled=yes
```

假设配置运行没有问题，您将看到一个大的 ASCII Asterisk 标志，然后在 Rocky Linux 上显示以下内容：

```bash
configure: Package configured for:
configure: OS type  : linux-gnu
configure: Host CPU : x86_64
configure: build-cpu:vendor:os: x86_64 : pc : linux-gnu :
configure: host-cpu:vendor:os: x86_64 : pc : linux-gnu :
```

### 设置 Asterisk 菜单选项 [更多选项]

这是管理员需要自行研究的关键步骤之一。有许多您可能不需要的菜单选项。运行以下命令：

```bash
make menuselect
```

将进入菜单选择界面：

![menuselect screen](../images/asterisk_menuselect.png)

仔细查看这些选项，根据您的需求进行选择。如前所述，这需要一些额外的研究。

### 构建并安装 Asterisk

要构建，请依次运行以下命令：

```bash
make
make install
```

安装文档不是必需的，但除非您是通信服务器专家，否则您会希望安装它们：

```bash
make progdocs
```

接下来，安装基本 PBX 并进行配置。基本 PBX 就是基本的功能，非常基础！您可能需要根据需求进行后续更改，以使您的 PBX 按预期运作。

```bash
make basic-pbx
make config
```

## Asterisk 配置

### 创建用户和组

您需要为 Asterisk 创建专用用户和组。现在创建它们：

```bash
groupadd asterisk
useradd -r -d /var/lib/asterisk -g asterisk asterisk
chown -R asterisk.asterisk /etc/asterisk /var/{lib,log,spool}/asterisk /usr/lib64/asterisk
restorecon -vr {/etc/asterisk,/var/lib/asterisk,/var/log/asterisk,/var/spool/asterisk}
```

由于大部分构建工作已完成，现在退出 `sudo -s` 命令。这意味着后续大部分命令需要再次使用 `sudo`：

```bash
exit
```

### 设置默认用户和组

```bash
sudo vi /etc/sysconfig/asterisk
```

取消以下两行的注释并保存：

```bash
AST_USER="asterisk"
AST_GROUP="asterisk"
```

```bash
sudo vi /etc/asterisk/asterisk.conf
```

取消以下两行的注释并保存：

```bash
runuser = asterisk ; The user to run as.
rungroup = asterisk ; The group to run as.
```

### 配置 Asterisk 服务

```bash
sudo systemctl enable asterisk
```

### 配置防火墙(firewall)

本例使用 `firewalld` 作为防火墙，这是 Rocky Linux 的默认防火墙。目标是向外界开放 SIP 端口，并根据 Asterisk 文档建议，在端口 10000-20000 范围内向外界开放 RTP (Realtime Transport Protocol，实时传输协议)。

您几乎肯定需要为其他面向公网的服务（HTTP/HTTPS）添加其他防火墙规则，这些服务可能只需要对您的 IP 地址开放。这超出了本文档的范围：

```bash
sudo firewall-cmd --zone=public --add-service sip --permanent
sudo firewall-cmd --zone=public --add-port=10000-20000/udp --permanent
```

由于 `firewalld` 命令已设为永久，需要重启服务器。可以使用以下命令：

```bash
sudo shutdown -r now
```

## 测试

### Asterisk 控制台

要测试，请连接到 Asterisk 控制台：

```bash
sudo asterisk -r
```

这将带您进入 Asterisk 命令行客户端。在显示基本 Asterisk 信息后，您将看到此提示符：

```bash
asterisk*CLI>
```

要更改控制台的详细程度，请使用以下命令：

```bash
core set verbose 4
```

这将在 Asterisk 控制台中显示以下内容：

```bash
Console verbose was OFF and is now 4.
```

### 显示示例终端认证信息

在 Asterisk 命令行客户端提示符下，输入：

```bash
pjsip show auth 1101
```

这将返回可用于连接任何 SIP 客户端(SIP client)的用户名和密码信息。

## 结论

以上内容将帮助您启动并运行服务器，但您需要负责完成配置、连接设备以及进一步的故障排除。

运维 Asterisk 通信服务器需要时间和精力，并且需要管理员进行研究。有关配置和使用 Asterisk 的更多信息，请参阅 [Asterisk Wiki](https://docs.asterisk.org/Configuration/)。
