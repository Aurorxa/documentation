---
title: 安装 Nerd Fonts
author: Franco Colussi
contributors: Steven Spencer, Ganna Zhyrnova, Christine Belzie
tested: 8.6, 9.0
tags:
    - nvchad
    - coding
    - fonts
---

# :material-format-font: Nerd Fonts - 面向开发者的字体

## :material-format-font: 什么是 Nerd Fonts？

![Nerd Fonts](images/nerd_fonts_site_small.png){ align=right } Nerd Fonts 是一系列修改过的字体，面向开发者。特别是，"图标字体"如 [Font Awesome](https://fontawesome.com/)、[Devicons](https://devicon.dev/) 和 [Octicons](https://primer.style/foundations/icons) 被用来添加额外的字形。

Nerd Fonts 还采用最流行的编程字体，如 MonoLisa 或 SourceCode Pro，并通过添加一组字形（图标）来修改它们。如果你想要使用的字体尚未被编辑，还可以使用字体补丁器。还有一个预览功能，可以查看字体在编辑器中的外观。请查看该项目的[主站点](https://www.nerdfonts.com/)了解更多信息。

## :material-monitor-arrow-down-variant: 下载

字体可从以下位置下载：

```text
https://www.nerdfonts.com/font-downloads
```

### :material-monitor-arrow-down-variant: 安装过程

在 Rocky Linux 中安装 Nerd Fonts 完全通过命令行完成，这得益于项目仓库 [ryanoasis/nerd-fonts](https://github.com/ryanoasis/nerd-fonts) 提供的流程实现。该流程使用 *git* 获取所需字体，并使用 *fc-cache* 进行配置。

!!! Note

    此方法可用于所有使用 [fontconfig](https://www.freedesktop.org/wiki/Software/fontconfig/) 进行系统字体管理的 *linux* 发行版。

开始时，从项目仓库获取必要的文件：

```bash
git clone --filter=blob:none --sparse git@github.com:ryanoasis/nerd-fonts
```

此命令仅下载必要的文件，省略 *patched-fonts* 中包含的字体，以免用以后不会使用的字体加重本地仓库负担，从而实现选择性安装。  
本指南将使用 [IBM Plex Mono](https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/IBMPlexMono) 字体，它提供干净且略带印刷感的显示效果，这些特性使其特别适合编写 Markdown 文档。  
访问[专用网站](https://www.programmingfonts.org/#plex-mono)了解可用字体的概述和预览。

转到新创建的文件夹，然后使用以下命令下载字体集：

```bash
cd ~/nerd-fonts/
git sparse-checkout add patched-fonts/IBMPlexMono
```

该命令将字体下载到 *patched-fonts* 文件夹，完成后你可以使用提供的 ==install.sh== 脚本安装它们，输入：

```bash
./install.sh IBMPlexMono
```

!!! Note "保留名称"

    该字体在安装期间被重命名为 *BlexMono*，以遵守 SIL Open Font License (OFL)，特别是[保留名称机制](http://scripts.sil.org/cms/scripts/page.php?item_id=OFL_web_fonts_and_RFNs#14cbfd4a)。

*install.sh* 脚本将字体复制到用户文件夹 `~/.local/share/fonts/`，并调用 *fc-cache* 程序在系统中注册它们。完成后，字体将可用于终端模拟器；特别是，我们将找到以下已安装的字体：

```text title="~/.local/share/fonts/"
NerdFonts/
├── BlexMonoNerdFont-BoldItalic.ttf
├── BlexMonoNerdFont-Bold.ttf
├── BlexMonoNerdFont-ExtraLightItalic.ttf
├── BlexMonoNerdFont-ExtraLight.ttf
├── BlexMonoNerdFont-Italic.ttf
├── BlexMonoNerdFont-LightItalic.ttf
├── BlexMonoNerdFont-Light.ttf
├── BlexMonoNerdFont-MediumItalic.ttf
├── BlexMonoNerdFont-Medium.ttf
├── BlexMonoNerdFont-Regular.ttf
├── BlexMonoNerdFont-SemiBoldItalic.ttf
├── BlexMonoNerdFont-SemiBold.ttf
├── BlexMonoNerdFont-TextItalic.ttf
├── BlexMonoNerdFont-Text.ttf
├── BlexMonoNerdFont-ThinItalic.ttf
├── BlexMonoNerdFont-Thin.ttf
```

## :material-file-cog-outline: 配置

此时，你选择的 Nerd Font 应该可供选择。要选择它，你必须参考你正在使用的桌面环境。

![Font Manager](images/font_nerd_view.png)

如果你使用的是 Rocky Linux 默认桌面（Gnome），要更改终端模拟器中的字体，只需打开 `gnome-terminal`，转到"首选项"，并为你的配置文件设置 Nerd Font。
