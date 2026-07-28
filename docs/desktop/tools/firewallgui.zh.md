---
title: 防火墙 GUI 应用
author: Ezequiel Bruni
contributors: Steven Spencer, Ganna Zhyrnova
---

## 简介

想要不通过命令行来管理防火墙吗？有一款专为 `firewalld`（Rocky Linux 中使用的防火墙）构建的优秀应用，可以通过 Flathub 获取。本指南将展示如何快速启动和运行它，以及界面的基础知识。

我们不会涵盖 `firewalld` 或 GUI 的全部功能，但应足以让您入门。

## 前提条件

本指南假定您具备以下条件：

* 安装了任一图形桌面环境的 Rocky Linux
* `sudo` 或管理员访问权限
* 对 `firewalld` 工作原理的基本了解

!!! note

    请记住，虽然如果您偏好使用 GUI 这款应用会让您更轻松，但您仍然需要理解 `firewalld` 的基本概念。您必须了解端口、区域 (Zones)、服务 (Services)、来源 (Sources) 等。

    如果对这些内容不清楚，请参阅 [`firewalld` 初学者指南](../../guides/security/firewalld-beginners.md)，特別阅读关于区域的内容，以了解它们的作用。

## 安装应用

进入"软件中心"(Software Center)应用，搜索 "Firewall"。它是 Rocky Linux 仓库中的一个原生软件包，名为 "Firewall"，应该很容易找到。

![软件中心中的 Firewall](images/firewallgui-01.png)

它在仓库中的名称是 `firewall-config`，可以用常规命令安装：

```bash
sudo dnf install firewall-config
```

打开应用时，它会询问密码。在执行敏感操作之前也会再次询问。

## 配置模式

首先要了解的是配置模式，可在窗口顶部的下拉菜单中选择。选项有 Runtime（运行时）和 Permanent（永久）。

![配置模式下拉菜单位于窗口顶部附近](images/firewallgui-02.png)

在 Runtime 模式下打开端口、添加允许的服务或进行其他任何更改都是*临时的*，且无法访问所有功能。重启或手动重新加载防火墙后，这些更改将消失。这在只需快速更改以完成单项任务或想在将其永久化之前测试更改时非常方便。

例如，在 Public（公共）区域中打开了一个端口后，可以转到 `Options > Runtime To Permanent` 来保存更改。

Permanent（永久）模式风险更高，但会开放所有功能。它允许创建新区域、单独配置服务、管理网络接口，以及添加 IPSet（即被允许或不被允许与您的计算机或服务器通信的 IP 地址集合）。

进行永久更改后，转到 `Options > Reload Firewalld` 以正确启用它们。

## 管理接口/连接

最左侧面板标记为 "Active Bindings"，这里可以找到网络连接并手动添加网络接口。如果向上滚动，您会看到我的以太网连接 (eno1)。"public" 区域默认受到良好保护并包含您的网络连接。

在面板底部，有 "Change Zone" 按钮，可以将连接分配到其他区域。在 Permanent 模式下，还可以创建自己的自定义区域 (Custom Zones)。

![截图展示窗口左侧的 Active Bindings 面板](images/firewallgui-03.png)

## 管理区域

在右侧面板的第一个标签页中，可以找到区域 (Zone) 菜单。在这里可以打开和关闭端口、启用或禁用服务、为入站流量添加受信任的 IP 地址（例如本地网络）、启用端口转发、添加富规则 (rich rules) 等。

对于大多数基础桌面用户来说，您将在该面板的 Services 和 Ports 子标签页中花费最多时间。

!!! Note

    从仓库安装应用和服务。其中一些（通常是为桌面使用设计的）会自动启用相关服务或打开相应端口。但是，如果没有生效，可以按照以下步骤手动完成。

### 向区域添加服务

服务是指 `firewalld` 默认支持的热门应用和后台服务。可以通过滚动列表并勾选相应复选框来快速轻松地启用它们。

例如，如果已安装 KDE Connect* 以帮助同步桌面与其他设备，现在希望允许它通过防火墙使其正常工作，可以：

1. 首先选择要编辑的区域。本例中使用默认的 public 区域。
2. 向下滚动列表，勾选 "kdeconnect"。
3. 如果处于 Runtime 配置模式，不要忘记在选项菜单中点击 "Runtime To Permanent" 和 "Reload Firewalld"。

\* 可从 EPEL 仓库获取。

![截图展示右侧面板中的 Zones 标签页和 Services 子面板](images/firewallgui-04.png)

列表中其他热门服务包括：用于托管网站的 HTTP & HTTPS、用于允许从其他设备进行终端访问的 SSH、用于托管 Windows 兼容文件共享的 Samba 等等。

但是，并非每个程序都在列表中，您可能需要手动打开端口。

### 在区域上打开端口

为特定应用打开端口很简单。只需阅读文档了解所需端口即可。

1. 再次选择要编辑的区域。
2. 转到右侧面板中的 Ports 标签页。
3. 点击 Add 按钮。
4. 在文本字段中填写需要打开的端口。确认应用需要的协议及其使用的网络协议（如 TCP/UDP 等）。
5. 点击 OK，然后使用 "Runtime To Permanent" 和 "Reload Firewalld" 选项。

![截图展示 Ports 子面板和用于输入端口号的弹窗](images/firewallgui-05.png)

## 结语

如果想锻炼大脑，应该阅读更多关于 `firewalld` 基础原理的内容。还可以使用右侧面板顶部的 "Services" 标签页（紧邻 "Zones"）来精确定义服务的工作方式，或通过 IPSet 和 Sources 控制允许与您计算机通信的其他计算机的访问权限。

或者打开 Jellyfin 服务器的端口然后继续做事。`firewalld` 是一个非常强大的工具，Firewall 应用可以帮助您以对初学者友好的方式探索其功能。
