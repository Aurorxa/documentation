---
title: Jellyfin 流媒体服务器
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - jellyfin
  - 媒体服务器
  - 流媒体
---

# Jellyfin 流媒体服务器

## 引言

Jellyfin 是一款免费开源的多媒体服务器，允许您从任何设备流式传输媒体文件到其他设备。它是 Emby 和 Plex 等专有解决方案的替代品。使用 Jellyfin，您可以在个人服务器上托管您的电影、电视节目、音乐等，并随时随地进行流式传输。

Jellyfin 的功能包括：

* 可自定义的元数据收集
* 直通或转码播放
* 多个用户和权限管理
* 媒体库组织
* 支持直播电视和 DVR 功能
* 广泛支持的客户端：拥有适用于各种平台（Web、Android、iOS、Roku 等）的客户端应用

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够在没有 root 权限的情况下执行命令的能力（通过 `sudo` 配置的用户）
* 熟悉命令行操作
* 具备使用 `vi`、`nano` 等命令行编辑器的经验
* 已配置防火墙

## 安装 EPEL，启用 PowerTools/CRB

首先，安装 EPEL（Extra Packages for Enterprise Linux）仓库：

```bash
dnf install epel-release
```

根据您的 Rocky Linux 版本启用 PowerTools 或 CRB 仓库：

Rocky Linux 8：

```bash
dnf config-manager --set-enabled powertools
```

Rocky Linux 9：

```bash
dnf config-manager --set-enabled crb
```

安装 `rpmfusion-free-release` 仓库。此步骤需要下载，在浏览器中访问 [rpmfusion](https://rpmfusion.org/Configuration)，然后根据您的 Rocky Linux 版本选择对应的发布版本进行安装：

```bash
dnf install --nogpgcheck https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm
```

!!! note

    将命令中的 `$(rpm -E %rhel)` 替换为您的 Rocky Linux 版本对应的数字。

安装 Jellyfin。首先添加 Jellyfin 仓库：

```bash
dnfo config-manager --add-repo https://repo.jellyfin.org/rocky/jellyfin.repo
```

然后安装 jellyfin：

```bash
dnf install jellyfin
```

## 配置 Jellyfin

安装后，启动并启用 Jellyfin：

```bash
systemctl enable --now jellyfin
```

Jellyfin 的 Web 管理界面默认运行在 `http://server_ip:8096`。在首次访问时，会要求您完成初始设置，包括创建管理员用户和添加库。

### 设置媒体目录

首先，创建媒体文件的存放目录：

```bash
mkdir -p /srv/media/{movies,tvshows,music}
```

确保 Jellyfin 服务有权限访问这些目录：

```bash
chown -R jellyfin:jellyfin /srv/media
```

### 配置防火墙

允许访问 Web 界面：

```bash
firewall-cmd --add-port=8096/tcp --permanent
firewall-cmd --reload
```

## 访问 Jellyfin

打开 Web 浏览器并导航到 `http://server_ip:8096`。首次访问时，系统将引导您完成初始设置：

1. 选择语言。
2. 创建管理员用户。
3. 添加媒体库，指向您已设置好的媒体目录。
4. 完成初始配置。

## 客户端

您可以使用多种客户端访问 Jellyfin 服务器，包括 Web 浏览器、Jellyfin 官方应用（适用于 Android、iOS、Apple TV、Android TV、Roku、Amazon Fire TV 等）以及第三方客户端。

## 结论

Jellyfin 提供了一个功能齐全、开源的媒体解决方案，您可以完全控制自己的媒体内容。无论您是将它用于个人收藏还是与家人朋友分享，它都是 Emby 和 Plex 等专有解决方案的绝佳替代品。
