---
title: Decibels 音频播放器
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - gnome
  - desktop
  - audio
  - flatpak
---

## 介绍

**Decibels** 是 GNOME 桌面的一款现代优雅的音频播放器。它建立在简洁的理念之上，旨在出色地完成一件事：播放音频文件。

与 Rhythmbox 等全功能的音乐库应用程序不同，Decibels 不管理音乐收藏。相反，它专注于为播放单个声音文件提供干净、直接的体验。其标志性功能是精美的波形显示，可以轻松、精确地在音频轨道中导航。

这使其成为无需将文件导入库即可快速收听下载的播客、语音备忘录或新歌的理想工具。

## 安装

在 Rocky Linux 上安装 Decibels 的推荐方式是作为 Flatpak 从 Flathub 仓库安装。此方法确保您拥有最新版本的应用程序，并与系统的其余部分隔离运行。

### 1. 启用 Flathub

首先，请确保 Flatpak 已安装并且 Flathub 远程已在您的系统上配置。

```bash
# 安装 Flatpak 包
sudo dnf install flatpak

# 添加 Flathub 远程仓库
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

!!! note
    您可能需要注销并重新登录，Flatpak 应用程序才会出现在 GNOME Activities Overview 中。

### 2. 安装 Decibels

启用 Flathub 后，您可以通过一条命令安装 Decibels：

```bash
flatpak install flathub org.gnome.Decibels
```

## 基本用法

安装后，您可以通过在 GNOME Activities Overview 中搜索"Decibels"来启动它。

要播放文件：

1.  启动应用程序。一个干净、简单的窗口将欢迎您。
2.  点击窗口中央突出的 **"Open a File..."** 按钮。
3.  使用文件选择器导航到并选择您系统上的音频文件（例如 `.mp3`、`.flac`、`.ogg` 或 `.wav` 文件）。
4.  文件将打开，并显示其波形。播放将自动开始。

## 主要功能

虽然简单，但 Decibels 具有几个有价值的功能：

*   **波形导航：** 而不是简单的进度条，Decibels 显示音频的波形。您可以点击波形上的任意位置，立即跳转到轨道的该部分。
*   **播放速度控制：** 右下角的控件允许您调整播放速度，非常适合加速播客或减慢音频以进行转录。
*   **快速跳过按钮：** 专用按钮允许您以 10 秒的间隔向后或向前跳转，使重新收听错过的短语变得容易。

对于任何需要在 GNOME 桌面上播放单个音频文件的简单、优雅和现代应用程序的人来说，Decibels 是一个绝佳的选择。
