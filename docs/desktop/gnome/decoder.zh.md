---
title: Decoder 二维码工具
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - gnome
  - desktop
  - qr code
  - flatpak
---

## 扫描与生成二维码

**Decoder** 是一款专为 GNOME 桌面设计的简洁优雅工具，其唯一目的是处理二维码。在这个二维码无处不在的时代——从分享 Wi-Fi 密码到访问餐厅菜单，拥有一款专门处理二维码的工具至关重要。

Decoder 在清晰专注的界面中提供两项主要功能：

1. **扫描：** 通过电脑摄像头或图像文件解码二维码。
2. **生成：** 根据您提供的任意文本创建二维码。

它与 GNOME 桌面的紧密集成使得它像是操作系统的原生组成部分。

## 安装

在 Rocky Linux 上推荐的安装方式是使用 Flathub 仓库提供的 Flatpak。这种方式可以確保您在安全沙箱环境中获得最新版本。

### 1. 启用 Flathub

如果您尚未完成此操作，请确保系统已安装 Flatpak 并配置了 Flathub 远程仓库。

```bash
# Install the Flatpak package
sudo dnf install flatpak

# Add the Flathub remote repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

### 2. 安装 Decoder

Flathub 启用后，只需一行命令即可安装 Decoder：

```bash
flatpak install flathub com.belmoussaoui.Decoder
```

## 如何使用 Decoder

安装完成后，可以从 GNOME "活动概览"(Activities Overview) 启动 Decoder。

### 扫描二维码

首次打开 Decoder 时，它已准备好进行扫描。您有两个选项：

* **摄像头扫描：** 点击左上角的摄像头图标。将出现一个窗口，显示您的摄像头画面。将摄像头对准二维码即可实时扫描。
* **从图像扫描：** 点击右上角的图像图标。这将打开文件选择器，允许您选择包含二维码的已保存图片或截图。

扫描到二维码后，Decoder 会智能地解析其内容。如果其中包含网站 URL，它将显示该链接并提供一个按钮，可在默认浏览器中打开。如果是纯文本内容，则会显示文本并提供一个便捷的按钮，可将内容复制到剪贴板。

### 生成二维码

要创建自己的二维码，请点击 Decoder 窗口顶部的"Generate"按钮。

1. 将出现一个文本框。在此框中输入或粘贴要编码的文字。
2. 随着输入，右侧会实时生成代表您文本的二维码。
3. 然后您可以点击 **"Save as Image..."** 按钮将二维码保存为 `.png` 文件，或点击 **"Copy to Clipboard"** 按钮将其粘贴到其他应用程序中。

Decoder 是 GNOME 设计理念的典范：简单、美观且高效的工具，将一件事做到极致。
