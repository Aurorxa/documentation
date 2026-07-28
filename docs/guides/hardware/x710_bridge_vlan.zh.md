---
title: 在 Intel X710 系列网卡上启用 VLAN 透传
author: Neel Chauhan
contributors: Ganna Zhyrnova
tested_with: 9.4
tags:
  - hardware
---

## 简介

某些服务器配备了 Intel X710 系列网络接口卡（NIC），例如作者的 Minisforum MS-01，该设备用于虚拟化防火墙。不幸的是，Rocky Linux 自带的驱动存在一个[已知问题](https://community.intel.com/t5/Ethernet-Products/X710-strips-incoming-vlan-tag-with-SRIOV/m-p/551464)，即 VLAN 无法按预期通过桥接接口透传。这种情况就发生在作者的 MikroTik CHR 虚拟机上。不过，这个问题是可以修复的。

## 前置条件与假设

使用本操作流程的最低要求如下：

* 一台配备 Intel X710 系列 NIC 的 Rocky Linux 8 或 9 服务器

## 安装 Intel 提供的 NIC 驱动

虽然 Rocky Linux 自带的驱动不支持 VLAN 透传，但 Intel 提供的驱动支持。首先，导航到 [Intel 驱动下载页面](https://www.intel.com/content/www/us/en/download/18026/intel-network-adapter-driver-for-pcie-40-gigabit-ethernet-network-connections-under-linux.html)。

![Intel X710 驱动下载页面](../images/intel_x710_drivers.png)

进入上述页面后，下载 `i40e_RPM_Files.zip` 文件，然后执行 `unzip`：

    unzip i40e_RPM_Files.zip

您将看到一系列 RPM 文件：

    kmod-i40e-2.25.11-1.rhel8u10.src.rpm
    kmod-i40e-2.25.11-1.rhel8u10.x86_64.rpm
    kmod-i40e-2.25.11-1.rhel8u7.src.rpm
    kmod-i40e-2.25.11-1.rhel8u7.x86_64.rpm
    kmod-i40e-2.25.11-1.rhel8u8.src.rpm
    kmod-i40e-2.25.11-1.rhel8u8.x86_64.rpm
    kmod-i40e-2.25.11-1.rhel8u9.src.rpm
    kmod-i40e-2.25.11-1.rhel8u9.x86_64.rpm
    kmod-i40e-2.25.11-1.rhel9u1.src.rpm
    kmod-i40e-2.25.11-1.rhel9u1.x86_64.rpm
    kmod-i40e-2.25.11-1.rhel9u2.src.rpm
    kmod-i40e-2.25.11-1.rhel9u2.x86_64.rpm
    kmod-i40e-2.25.11-1.rhel9u3.src.rpm
    kmod-i40e-2.25.11-1.rhel9u3.x86_64.rpm
    kmod-i40e-2.25.11-1.rhel9u4.src.rpm
    kmod-i40e-2.25.11-1.rhel9u4.x86_64.rpm

要安装的文件格式为 `kmod-i40e-2.25.11-1.rhelXuY.x86_64.rpm`，其中 `X` 和 `Y` 分别是 Rocky Linux 的主版本号和次版本号。例如，在作者的 Rocky Linux 9.4 服务器上，`X` 为 9，`Y` 为 4，因此安装包为：

    sudo dnf install kmod-i40e-2.25.11-1.rhel9u4.x86_64.rpm

安装驱动后，需要重启服务器：

    sudo reboot

重启后，X710 NIC 应能够通过桥接接口透传 VLAN。
