---
title: 使用 AppImagePool 安装 AppImage
author: Joseph Brinkman
contributors: Steven Spencer
---

## 简介

[AppImagePool](https://github.com/prateekmedia/appimagepool) 提供了一个用于安装和管理 AppImage 的中心。它的界面与 Software 应用类似。

## 前提条件

本指南需要以下条件：

* 安装了桌面环境的 Rocky Linux
* `sudo` 权限
* 系统上安装了 Flatpak

## 安装 AppImagePool

安装 AppImagePool 的 Flatpak 包：

```bash
flatpak install flathub io.github.prateekmedia.appimagepool
```

## 探索 AppImage 启动器

AppImagePool 安装完成后，启动它并探索可用的 AppImage。

![启动 AppImagePool](images/appimagepool/appimagepool_launch.jpg)

截至本文撰写时，共有 18 个可用类别：

1. 工具 (Utility)
2. 网络 (Network)
3. 图形 (Graphics)
4. 系统 (System)
5. 科学 (Science)
6. 其他 (Others)
7. 开发 (Development)
8. 游戏 (Game)
9. 教育 (Education)
10. 办公 (Office)
11. 多媒体 (Multimedia)
12. 音频 (Audio)
13. 模拟器 (Emulator)
14. 财务 (Finance)
15. Qt
16. 视频 (Video)
17. GTK
18. 音序器 (Sequencer)

此外，还有一个"探索"类别，用于浏览所有可用的 AppImage 类别。

## 下载 AppImage

找到您想使用的 AppImage：

![选择 AppImage](images/appimagepool/appimagepool_select.jpg)

点击其缩略图并下载。稍等片刻后，AppImage 就会出现在您的系统上并随时可用！

![已下载的 AppImage](images/appimagepool/appimagepool_download.jpg)

## 移除 AppImage

要移除 AppImage，点击顶部菜单栏中的 ++"Installed"++，然后点击要移除的 AppImage 右侧的垃圾桶图标：

![移除 AppImage](images/appimagepool/appimagepool_remove.jpg)

## 结论

[AppImagePool](https://github.com/prateekmedia/appimagepool) 提供了一个易于使用的中心，用于浏览、下载和移除 AppImage。它的外观与 Software 中心类似，使用起来同样简单。
