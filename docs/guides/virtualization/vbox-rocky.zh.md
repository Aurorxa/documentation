---
title: VirtualBox 上的 Rocky Linux
author: Steven Spencer
contributors: Trevor Cooper, Ezequiel Bruni, Ganna Zhyrnova
tested on: 8.4, 8.5
tags:
  - virtualbox
  - virtualization
---

# VirtualBox&reg; 上的 Rocky Linux

## 简介

VirtualBox&reg; 是一款功能强大的虚拟化产品，适用于企业级和家庭使用。偶尔有人报告说在 VirtualBox&reg; 中运行 Rocky Linux 遇到问题。实际上，从候选发布版开始测试和运行 VirtualBox&reg; 就一切正常。人们通常报告的问题往往涉及视频显示。

本文档旨在提供一份在 VirtualBox&reg; 中安装和运行 Rocky Linux 的分步说明。编写本文档的机器运行的是 Linux，但你可以使用任何受支持的操作系统。

## 前提条件

* 一台有可用内存和硬盘空间来构建和运行 VirtualBox&reg; 实例的机器（Windows、Mac、Linux、Solaris）。
* 在你的机器上安装了 VirtualBox&reg;。你可以在[此处](https://www.virtualbox.org/wiki/Downloads)找到。
* 一份适用于你架构的 Rocky Linux [DVD ISO](https://rockylinux.org/download) 副本（x86_64 或 ARM64）。
* 确保你的操作系统是 64 位的，并且在 BIOS 中开启了硬件虚拟化。

!!! Note

    硬件虚拟化是安装 64 位操作系统的绝对必要条件。如果你的配置界面只显示 32 位选项，你必须先停下来解决这个问题再继续。

## 准备 VirtualBox&reg; 配置

安装 VirtualBox&reg; 后，下一步是启动它。在没有任何镜像安装的情况下，你会看到一个类似于这样的界面：

 ![VirtualBox 全新安装](../images/vbox-01.png)

 首先，你需要告诉 VirtualBox&reg; 你要使用的操作系统：

* 点击 "New" (新建)（锯齿图标）。
* 键入一个名称。例如："Rocky Linux 8.5"。
* 保持 Machine Folder (机器文件夹) 为自动填充。
* 将 Type (类型) 更改为 "Linux"。
* 选择 "Red Hat (64-bit)"。
* 点击 "Next" (下一步)。

 ![名称和操作系统](../images/vbox-02.png)

接下来，你需要为这台机器分配一些 RAM。默认情况下，VirtualBox&reg; 会自动将其填充为 1024 MB。这对于任何现代操作系统（包括 Rocky Linux）来说都不是最优的。如果你有富余内存，分配 2 到 4 GB（2048 MB 或 4096 MB）——或者更多。VirtualBox&reg; 只在虚拟机运行期间使用这些内存。

此项没有截图，只需根据你的可用内存更改数值。运用你的最佳判断。

你需要设置硬盘大小。默认情况下，VirtualBox&reg; 会自动选中 "Create a virtual hard disk now" (现在创建虚拟硬盘) 单选按钮。

![硬盘](../images/vbox-03.png)

* 点击 ++"Create"++ (创建)

你将看到一个用于创建各种虚拟硬盘类型的对话框。这里有几个硬盘类型。有关选择虚拟硬盘类型，请参阅 Oracle VirtualBox 文档的[更多信息](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/vdidetails.html)。对于本文档，保持默认选项 (VDI)：

![硬盘文件类型](../images/vbox-04.png)

* 点击 ++"Next"++ (下一步)

下一个界面涉及物理硬盘上的存储方式。有两个选项。"Fixed Size" (固定大小) 创建较慢，使用较快，但在空间方面灵活性较差（如果需要更多空间，不能超过创建时的容量）。

默认选项 "Dynamically Allocated" (动态分配) 创建较快，使用较慢，但可以在硬盘空间需要变化时进行增长。

![物理硬盘上的存储方式](../images/vbox-05.png)

* 点击 ++"Next"++ (下一步)

VirtualBox&reg; 现在让你选择虚拟硬盘文件的位置。还有一个选项可以扩展默认的 8 GB 虚拟硬盘空间。这个选项很好，因为 8 GB 硬盘空间不足以安装任何 GUI 安装选项，更不用说使用了。根据你打算用虚拟机做什么以及你有多少可用磁盘空间，将其设置为 20 GB（或更多）：

![文件位置和大小](../images/vbox-06.png)

* 点击 ++"Create"++ (创建)

你已经完成了基本配置。你将看到一个类似下图的界面：

![基本配置完成](../images/vbox-07.png)

## 挂载 ISO 镜像

下一步是将你下载的 ISO 镜像作为虚拟 CD ROM 设备挂载。点击 "Settings" (设置)（齿轮图标），你将看到如下界面：

![设置](../images/vbox-08.png)

* 点击左侧菜单中的 "Storage" (存储) 项。
* 在中间部分的 "Storage Devices" (存储设备) 下，点击显示 "Empty" (空) 的 CD 图标。
* 在右侧的 "Attributes" (属性) 下，点击 CD 图标。
* 选择 "Choose/Create a Virtual Optical Disk" (选择/创建虚拟光盘)。
* 点击 "Add" (添加) 按钮（加号图标）并导航到你 Rocky Linux ISO 镜像的位置。
* 选择 ISO 并点击 "Open" (打开)。

你的 ISO 现在应该已添加为可用设备，如下所示：

![ISO 镜像已添加](../images/vbox-09.png)

* 高亮选中 ISO 镜像，然后点击 "Choose" (选择)。

Rocky Linux ISO 镜像现在显示在中间部分的 "Controller:IDE" 下已选中：

![ISO 镜像已选中](../images/vbox-10.png)

* 点击 ++"OK"++ (确定)

### 图形化安装的视频内存

VirtualBox&reg; 默认分配 16 MB 内存用于视频显示。如果你计划运行不带 GUI 的裸机服务器，这是可以的，但一旦添加了图形界面，这就远远不够了。保持此设置的用户通常会看到永久卡住的启动画面，或其他错误。

如果运行带有 GUI 的 Rocky Linux，请分配足够的内存来运行图形界面。如果你的机器内存紧张，将此值从 16 MB 向上调整，直到运行流畅。你的宿主机显示分辨率也是需要考虑的一个因素。

仔细考虑你希望 Rocky Linux 虚拟机做什么，并尝试分配与你的宿主机和其他需求兼容的视频内存。你可以在 [Oracle 官方文档](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/settings-display.html)中找到有关显示设置的更多信息。

如果你有足够的内存，可以将此值设置为最大值 128 MB。要在启动虚拟机之前修复此问题，请点击 "Settings" (设置)（齿轮图标），你应该会看到与挂载 ISO 镜像时相同的设置界面（如上所述）。

这次：

* 点击左侧的 "Display" (显示)。
* 在右侧的 "Screen" (屏幕) 选项卡中，你会注意到 "Video Memory" (视频内存) 选项，默认设置为 16 MB。
* 将其更改为你想要的值。你可以随时回到此界面上调此值。在此示例中，它是 128 MB。

!!! Tip

    存在将视频内存设置为高达 256 MB 的方法。如果你需要更多，请查看 Oracle 官方文档中的[这份文档](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/vboxmanage-modifyvm.html)。

你的界面应该看起来像这样：

![视频内存设置](../images/vbox-12.png)

* 点击 ++"OK"++ (确定)

## 开始安装

你已经准备好了一切，可以开始安装了。注意，在 VirtualBox&reg; 机器上安装 Rocky Linux 与在独立硬件上安装没有特别差异。安装步骤是相同的。

现在你已经为安装准备好了一切，你需要点击 "Start" (启动)（绿色右箭头图标）来开始安装 Rocky Linux。在点击跳过语言选择界面后，你的下一个界面是 "Installation Summary" (安装概要)。你需要设置任何与你相关的项目，但以下项目是必须的：

* Time & Date (时间和日期)
* Software Selection (软件选择)（如果你想使用除默认 "Server with GUI" 以外的选项）
* Installation Destination (安装目标位置)
* Network & Hostname (网络和主机名)
* User Settings (用户设置)

如果你对这些设置中的任何一项不确定，请参阅[安装 Rocky Linux 的文档](../installation.md)。

安装完成后，你应该有一个运行中的 Rocky Linux VirtualBox&reg; 实例。

安装和重启后，你会看到一个需要同意的 EULA (最终用户许可协议) 界面。点击 "Finish Configuration" (完成配置) 后，你应该会看到一个图形界面（如果选择了 GUI 选项）或命令行登录界面。作者选择了默认的 "Server with GUI" 用于演示目的：

![运行中的 Rocky Linux VirtualBox 机器](../images/vbox-11.png)

## 其他信息

本文档并不打算让你成为 VirtualBox&reg; 所有功能的专家。有关如何执行特定操作的信息，请查看[官方文档](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/)。

!!! tip "高级提示"

    VirtualBox&reg; 通过 `VBoxManage` 在命令行提供了广泛的选项。虽然本文档不涵盖 `VBoxManage` 的使用，但如果你想进一步研究，Oracle 的官方文档提供了[大量详细信息](https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/vboxmanage-intro.html)。

## 结论

创建、安装和运行一台 VirtualBox&reg; Rocky Linux 机器很简单。虽然这不是一份详尽的指南，但按照上述步骤操作应该能让你获得一个运行中的 Rocky Linux 安装。如果你使用 VirtualBox&reg; 并有想分享的特定配置，作者邀请你提交本文档的新章节。
