---
title: Cockpit KVM 仪表盘
author: Neel Chauhan
contributors: Steven Spencer,Ganna Zhrynova
tested on: 9.3, 10.1
tags:
  - virtualization
---

# Cockpit KVM 仪表盘

## 简介

Cockpit 是一个服务器管理工具，提供一个易于使用的仪表盘来管理你的服务器。Cockpit 的一个功能是，通过一个软件包，它可以从 Web 界面管理 KVM (基于内核的虚拟机) 虚拟化虚拟机，类似于 VMware ESXi 或 Proxmox。

## 前提条件

* 一台启用了硬件虚拟化的 Rocky Linux 服务器
* 能够访问 Rocky Linux 的 `dnf` 仓库

## 安装 Cockpit

Cockpit 在 Rocky Linux 中默认自带。然而，KVM 支持并不会开箱即装。使用 `dnf` 安装它以及其他所需的软件包：

```bash
dnf install -y cockpit-machines cockpit-ws cockpit-system libvirt
```

## 启用 `cockpit`

要同时启用 KVM 虚拟化和 Cockpit，请启用 `systemd` 服务：

```bash
systemctl enable --now libvirtd cockpit.socket
```

启用 `cockpit` 后，在浏览器中打开 <http://ip_address:9090>（注意：将 **ip_address** 替换为你服务器的 IP 地址）：

![Cockpit 登录界面](../images/cockpit_login.png)

以非 root 用户身份登录，你应该会看到一个类似下图的仪表盘：

![Cockpit 仪表盘](../images/cockpit_dashboard.png)

## 创建虚拟机

在本指南中，你将在宿主机系统上创建一个 Rocky Linux 9 虚拟机，使用自动化方式添加用户名和 root 密码。

要在 Cockpit 中创建虚拟机，首先点击蓝色的 **Turn on administrative access** (开启管理访问权限) 按钮，并在需要时输入你的密码：

![Cockpit 以 root 身份登录的仪表盘](../images/cockpit_root_dashboard.png)

你现在已以 root 身份在 Cockpit 中登录。在侧边栏中，点击 **Virtual Machines** (虚拟机)：

![Cockpit 虚拟机仪表盘](../images/cockpit_vm_dashboard.png)

然后点击 **Create VM** (创建虚拟机)：

![虚拟机创建对话框](../images/cockpit_vm_create_1.png)

在 **Operating system** (操作系统) 下拉菜单中，选择 **Rocky Linux 9 (Blue Onyx)**：

![已选中 Rocky Linux 9 的虚拟机创建对话框](../images/cockpit_vm_create_2.png)

接下来，点击 **Automation** (自动化)，填写你想要在新虚拟机上使用的登录信息：

![已填写 root 密码和用户名的虚拟机创建对话框](../images/cockpit_vm_create_2.png)

最后，选择 **Create and run** (创建并运行)。

几分钟后，选择你新创建的虚拟机，你将看到它的 IP 地址：

![我们虚拟机的 IP 地址](../images/cockpit_vm_ip.png)

通过 SSH 连接到你的 hypervisor (虚拟机管理器)，然后通过 SSH 连接到 Cockpit 显示的 IP 地址。在这个例子中，它是 **172.20.0.103**。你将登录到你的新服务器：

![我们虚拟机的终端](../images/cockpit_vm_terminal.png)

## 局限性

虽然 Cockpit 在创建和管理虚拟机方面很棒，但有一些局限性需要了解：

* 你无法创建桥接接口。
* 你只能在 `default` 存储池中创建新镜像，不能在任意存储池中创建。

幸运的是，你可以在命令行创建这些，然后 Cockpit 就可以使用它们了。

## 结论

Cockpit 是通过 Web 界面管理 Rocky Linux 服务器的宝贵工具。就作者个人而言，它是他们的家实验室中创建虚拟机的首选工具。虽然 `cockpit-machines` 可能不如 ESXi 或 Proxmox 功能完整，但它能满足 90% 的 hypervisor 使用场景。
