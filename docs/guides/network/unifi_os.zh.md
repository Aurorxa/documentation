---
title: Ubiquiti UniFi OS 控制器
author: Neel Chauhan
contributors: Steven Spencer
tested_with: 10.1
tags:
  - network
  - network management
  - Ubiquiti
---

## 简介

Ubiquiti 的 UniFi 系列无线接入点和其他网络设备在小型企业和家庭实验室环境中非常受欢迎。这包括作者的家庭实验室。

UniFi OS Server 基于 Podman 设计，是下一代 UniFi 控制器软件。

虽然技术上为 Debian 和 Ubuntu 设计，但也可以在 Rocky Linux 上安装 Ubiquiti 的 UniFi OS Server。

## 前置条件

- 一台至少有 15GB 空闲磁盘空间的服务器或虚拟机
- 至少一个 Ubiquiti UniFi 设备在您的 LAN 上
- 网络管理知识
- 对某些命令具有提权（`sudo`）权限

## 安装前置软件包

安装前置软件包：

```bash
sudo dnf install -y podman slirp4netns
```

## 下载 UniFi OS Server

前往 [Ubiquiti 下载页面](https://ui.com/download) 并复制所需 CPU 架构的 UniFi OS 链接。

下载文件：

```bash
wget UNIFI_OS_SERVER_DOWNLOAD_LINK
```

作者的文件是 `1856-linux-x64-5.0.6-33f4990f-6c68-4e72-9d9c-477496c22450.6-x64`。使其可执行：

```bash
chmod a+x 1856-linux-x64-5.0.6-33f4990f-6c68-4e72-9d9c-477496c22450.6-x64
```

## 安装 UniFi OS Server

运行下载的文件：

```bash
./1856-linux-x64-5.0.6-33f4990f-6c68-4e72-9d9c-477496c22450.6-x64
```

您将收到一个安装提示。选择 `y`：

```text
You are about to install UOS Server version 5.0.6. Proceed? (y/N): y
```

安装完成后，禁用 `firewalld`：

```bash
sudo systemctl disable --now firewalld
```

您将看到一行输出：

```text
UOS Server is running at: https://IP:11443/
```

在浏览器中输入该地址。

自此以后的步骤应该是不言自明的。

## 总结

与 Ubiquiti 之前需要基于 Debian 发行版的 UniFi Network 控制器不同，UniFi OS 增加了在 Rocky Linux 上运行的能力。这使得它能够适用于那些已标准化企业 Linux 且不希望使用 UniFi 网关的环境。例如，作者使用 MikroTik 核心路由器和交换机，搭配 UniFi 接入点以实现更好的 Wi-Fi 覆盖。
