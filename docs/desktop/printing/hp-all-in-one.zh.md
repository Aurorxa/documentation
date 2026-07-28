---
title: HP 一体机打印机安装与设置
author: Joseph Brinkman
contributors: Steven Spencer, Ganna Zhyrnova
tested with: 9.4
tags:
  - desktop
  - printer support
---

## 简介

得益于 [HPLIP](https://developers.hp.com/hp-linux-imaging-and-printing/about){target="_blank"}，在 Linux 上使用 HP 一体机进行打印和扫描是可行的。

本指南使用 HP Deskjet 2700 系列进行了测试。

请查看 [所有支持的打印机](https://developers.hp.com/hp-linux-imaging-and-printing/supported_devices/index){target="_blank"} 以确认 HPLIP 包是否支持您的打印机。

## 下载与安装 HPLIP

HPLIP 是 HP 提供的第三方软件，包含必要的打印机驱动。安装以下 3 个软件包以获得完整的图形用户界面支持。

```bash
sudo dnf install hplip-common.x86_64 hplip-libs.x86_64 hplip-gui
```

## 打印机设置

驱动安装完成后，即可将 HP 一体机添加到您的 Rocky Workstation 中。确保打印机通过 Wi-Fi 或直连方式物理接入同一网络。进入"设置"(Settings)。

1. 在左侧菜单中，点击 ++"Printers"++

2. 点击 ++"Add a Printer"++

3. 选择您的 HP 一体机。

## 扫描仪支持

虽然可以使用 HPLIP 包中的 CLI 命令进行扫描，但它们不提供扫描仪应用程序。请安装 XSane，一款易于使用的扫描工具。

```bash
sudo dnf install sane-backends sane-frontends xsane
```

XSane 的图形界面看起来有些复杂，但简单扫描却很直接。启动 XSane 后，窗口中有按钮可以 ++"Acquire a preview"++（获取预览）。这将对扫描进行预览。准备好扫描后，在主菜单中点击 'Start'。

如需更全面的 XSane 指南，请阅读这篇 [剑桥大学数学学院文章](https://www.maths.cam.ac.uk/computing/printing/xsane){target="_blank"}。

## 结语

安装 HPLIP 和 XSane 后，您现在应该能够使用 HP 一体机进行打印和扫描。
