---
title: KDE 桌面安装
author: Steven Spencer
contributors: Ganna Zhyrnova
---

# KDE 安装步骤

## 前提条件

* Rocky Linux 完整安装（DVD），或者安装了其他桌面环境的系统。
* 能够以 `sudo` 用户身份执行命令。

## 引言

KDE 是一款运行在 Linux 上的功能齐全的桌面环境(Desktop Environment)，类似于 GNOME。它是 Rocky Linux 上可用的众多桌面环境之一。本文档将引导您在 Rocky Linux 上完成 KDE 的安装。

## 添加 EPEL 仓库

如果您第一次通过 DVD 完整安装 Rocky Linux，此步骤不是必需的；但如果您的系统上没有安装 EPEL，则需要添加它：

```bash
sudo dnf install epel-release
```

## 更新系统

```bash
sudo dnf update
```

## 安装 KDE

KDE 是作为组安装的，因此只需一个简单的命令即可完成安装：

```bash
sudo dnf group install "KDE Plasma Workspaces"
```

## 安装 SDDM

您需要一种方式来启动 KDE。为此，我们推荐安装 SDDM。SDDM 是一个现代的显示管理器(Display Manager)，适用于 X11 和 Wayland，我们选择它作为默认的管理器。要安装它：

```bash
sudo dnf install sddm sddm-kcm
```

接下来，切换到以下目标：

```bash
sudo systemctl set-default graphical.target
systemctl enable sddm
```

如果系统一直处于多用户模式（命令提示符），现在重新启动。重启后，SDDM 会让您选择 KDE 作为桌面环境。

## 配置 KDE

KDE 提供了广泛的配置选项。作为第一次接触 KDE 的用户，您可能会觉得它的界面与 Windows 相似。

Rocky Linux 的默认 KDE 界面非常简洁和美观，如下图所示：

![KDE 默认界面](../images/kde_first_screen.png)

KDE 拥有极其丰富的自定义功能，您可以对其外观、行为、工作区等进行深度定制。通过可视化配置工具，可以轻松调整各种效果的感官体验。您可以访问菜单中的设置：

![KDE 设置菜单](../images/kde_settings_menu.png)

!!! error "注意"

    某些设置的高级效果可能会对系统的稳定性产生影响。因此请谨慎调整，并建议在调整后进行测试。

您也可以借助 "在 Web 浏览器中打开"(Open in Web Browser)的功能，直接从 KDE 商店获取新的图像主题和图标，该功能可以让您一键安装新主题，然后看起来非常惊艳。例如，如果您选择主题后保存设置，可以看到新的外观和配色方案。您可以根据自己的喜好定制完全不同的外观。

## 结论

KDE 是一个适合那些喜欢在高度定制化桌面环境中工作的用户的优秀选择。它的功能丰富，安装简单，能够满足各种不同的需求。
