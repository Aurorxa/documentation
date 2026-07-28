---
title: 基于主机的入侵检测系统 (HIDS)
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - web
  - security
  - ossec-hids
  - hids
---

# 基于主机的入侵检测系统 (HIDS)

## 前提条件

* 熟练使用命令行文本编辑器（本示例使用 `vi`）
* 对在命令行中执行命令、查看日志以及其他一般系统管理员职责有较高的熟练度
* 理解安装此工具还需要监控操作并根据你的环境进行调优
* 所有命令由 root 用户或具有 `sudo` 权限的普通用户运行

## 简介

`ossec-hids` 是一个主机入侵检测系统，提供自动操作-响应步骤来帮助缓解主机入侵攻击。它只是一个加固 Apache Web 服务器设置中可能的一个组成部分。你可以单独使用它，也可以与其他工具一起使用。

如果你想使用此工具和其他加固工具，请参考 [Apache 加固 Web 服务器](index.md) 文档。本文档也使用了该原始文档中概述的所有假设和约定。在继续之前，建议你查看它。

## 安装 Atomicorp 仓库

要安装 `ossec-hids`，我们需要来自 Atomicorp 的第三方仓库。Atomicorp 也为那些希望在遇到问题时获得专业支持的人提供价格合理的付费支持版本。

如果你更倾向于有支持，并且预算允许，请查看 [Atomicorp 的付费 `ossec-hids`](https://atomicorp.com/atomic-enterprise-ossec/) 版本。你只需要来自 Atomicorp 无成本仓库的几个软件包。下载后你将要更改仓库配置。

下载仓库需要 `wget`。先安装它，如果你尚未安装 EPEL 仓库，也一并安装：

```bash
dnf install wget epel-release
```

下载并启用 Atomicorp 的无成本仓库：

```bash
wget -q -O - https://www.atomicorp.com/installers/atomic | sh
```

此脚本将要求你同意条款。输入 "yes" 或按 ++enter++ 接受默认值。

接下来，它会询问你是否希望默认启用该仓库，再次接受默认值或输入 "yes"。

### 配置 Atomicorp 仓库

你只需 atomic 仓库中的几个软件包。因此，你将更改仓库并仅指定所需的软件包：

```bash
vi /etc/yum.repos.d/atomic.repo
```

在顶部区域 "enabled = 1" 的下一行添加此内容：

```bash
includepkgs = ossec* GeoIP* inotify-tools
```

这是你唯一需要做的更改。保存你的更改并退出仓库文件（在 `vi` 中，按 ++esc++ 进入命令模式，然后按 ++shift+colon+"wq"++ 保存并退出）。

这将 Atomicorp 仓库限制为只能安装和更新这些软件包。

## 安装 `ossec-hids`

配置好仓库后，你需要安装这些软件包：

```bash
dnf install ossec-hids-server ossec-hids inotify-tools
```

### 配置 `ossec-hids`

默认配置处于需要大量更改的状态。其中大部分与服务器管理员通知和日志位置有关。

`ossec-hids` 查看日志，尝试判断是否有攻击正在进行，以及是否应用缓解措施。它还会向服务器管理员发送报告，包含通知或关于基于 `ossec-hids` 所见而启动的缓解措施的消息。

要编辑配置文件，输入：

```bash
vi /var/ossec/etc/ossec.conf
```

作者将拆解此配置，逐行显示更改并解释：

```bash
<global>
  <email_notification>yes</email_notification>  
  <email_to>admin1@youremaildomain.com</email_to>
  <email_to>admin2@youremaildomain.com</email_to>
  <smtp_server>localhost</smtp_server>
  <email_from>ossec-webvms@yourwebserverdomain.com.</email_from>
  <email_maxperhour>1</email_maxperhour>
  <white_list>127.0.0.1</white_list>
  <white_list>192.168.1.2</white_list>
</global>
```

电子邮件通知默认是关闭的，`<global>` 配置几乎是空的。你需要开启电子邮件通知，并通过电子邮件地址识别将接收电子邮件报告的人员。

`<smtp_server>` 部分当前显示 localhost，然而如果你愿意，可以指定一个电子邮件服务器中继，或者按照[本指南](../../email/postfix_reporting.md)为本地主机设置 postfix 邮件设置。

你需要设置 "from" 电子邮件地址。你需要这样做来应对你的邮件服务器上的 SPAM (垃圾邮件) 过滤器，它可能将这些电子邮件视为垃圾邮件。为避免被电子邮件淹没，将电子邮件报告设置为每小时 1 封。在开始使用 `ossec-hids` 时，你可以扩展或注释掉此命令。

`<white_list>` 部分涉及服务器的 localhost IP 以及防火墙的"公网" IP 地址（记住我们用的是私有 IP 地址替代），所有来自可信网络的连接都将显示为该地址。你可以添加许多 `<white_list>` 条目。

```bash
<syscheck>
  <!-- syscheck 的执行频率 -- 默认每 22 小时一次 -->
  <frequency>86400</frequency>
...
</syscheck>
```

`<syscheck>` 部分查看在查找被入侵文件时要包含和排除的目录列表。可以将其视为另一个用于监视和保护文件系统免受漏洞攻击的工具。你需要查看目录列表，并将你想要的其他目录添加到 `<syscheck>` 部分。

紧接在 `<syscheck>` 部分下方的 `<rootcheck>` 部分是另一个保护层。`<syscheck>` 和 `<rootcheck>` 监视的位置是可编辑的，但你可能不需要对它们做任何更改。  

将 `<rootcheck>` 的运行 `<frequency>` (频率) 从默认的 22 小时更改为每 24 小时一次（86400 秒）是一个可选的更改，如上所示。

```bash
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/httpd/*access_log</location>
</localfile>
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/httpd/*error_log</location>
</localfile>
```

`<localfile>` 部分涉及你想要监视的日志的位置。_syslog_ 和 _secure_ 日志的条目已经存在，你只需验证其路径，但其他所有内容可以保持不变。

你需要添加 Apache 日志位置，并且希望将它们作为通配符添加，因为你可能有许多不同 Web 客户的日志。

```bash
  <command>
    <name>firewalld-drop</name>
    <executable>firewall-drop.sh</executable>
    <expect>srcip</expect>
  </command>

  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <level>7</level>
  </active-response>
```

最后，在文件末尾附近，你需要添加主动响应部分。这有两个部分：`<command>` 部分和 `<active-response>` 部分。

"firewall-drop" 脚本已经存在于 `ossec-hids` 路径中。它告诉 `ossec-hids`，如果发生级别 7 的事件，就添加一个防火墙规则来阻止该 IP 地址。

当所有配置更改完成后，启用并启动服务。如果一切正常启动，你就可以继续了：

```bash
systemctl enable ossec-hids
systemctl start ossec-hids
```

`ossec-hids` 配置文件。你可以通过访问[官方文档站点](https://www.ossec.net/docs/)了解这些选项。

## 结论

`ossec-hids` 只是 Apache 加固 Web 服务器的一个组成部分。通过将其与其他工具结合选择，你可以获得更好的安全性。

虽然安装和配置相对简单，但你会发现这并 **不是** 一个"装上就不管"的应用程序。你需要根据你的环境对其进行调优，以获得最大的安全性和最少的误报。
