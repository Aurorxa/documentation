---
title: 使用 AppImage 安装软件
author: Joseph Brinkman
contributors: Steven Spencer, Ganna Zhyrnova
---

## 简介

AppImage 是在 Linux 上安装软件的一种便捷方式，无需使用包管理器或命令行。它们是包含程序所有依赖项的单文件可执行文件，使其易于在各种 Linux 发行版上运行。对于熟悉 Windows 和 Mac 操作系统的最终用户来说，使用 AppImage 安装软件可能比管理仓库或从源码构建更为直观。

在 Rocky Linux 桌面上使用 AppImage 安装程序分为三个步骤：

1. 下载所需程序的 AppImage
2. 使程序可执行
3. 运行程序以安装它

本指南使用的示例程序是 Krita。您将使用 AppImage 下载并安装它。Krita 是一款免费且开源的图形设计软件。由于本指南主要讲解 AppImage 的功能，因此不会详细介绍 Krita 的使用方法。您可以[在其网站上了解更多信息](https://krita.org/)。

## 前提条件

本指南需要以下条件：

* 安装了桌面环境的 Rocky Linux
* `sudo` 权限

## 下载程序的 AppImage

使用 AppImage 安装软件的第一步是下载程序的 AppImage。要下载 Krita AppImage，请前往[下载页面](https://krita.org/en/download/)并点击 `Download` 按钮。

![点击下载 AppImage 按钮](images/download_krita_appimage.webp)

## 使用 AppImage 安装程序

下载 AppImage 后，您需要进入 `Downloads` 文件夹，使文件可执行后再运行它。

在 Rocky Linux 桌面的左上角，点击 Activities：

![Rocky Linux 桌面默认壁纸，鼠标悬停在 Activities 按钮上](images/activites_appimage.webp)

Activities 面板启动后，在搜索框中输入 'files'。点击 Files 应用：

![Rocky Linux 系统的 Activities 面板，搜索框中输入了 'Files'，鼠标悬停在 Files 应用上](images/searchbar_files_appimage.webp)

Files 将在主目录中打开。点击 Downloads 文件夹：

![Files 应用已打开，鼠标悬停在 Downloads 文件夹上](images/files_downloads_appimage.webp)

现在您已经进入了 AppImage 所在的目录，接下来需要使程序可执行。右键单击 AppImage 文件并选择 Properties：

![AppImage 文件已选中，鼠标悬停在 Properties 上](images/file_properties_appimage.webp)

在文件属性菜单中选择 Permissions：

![文件属性菜单中已选中 Permissions](images/permissions_appimage.webp)

勾选标签为 'Execute' 的复选框，然后关闭属性菜单：

![已勾选标签为 'Execute' 的复选框](images/file_properties_allow_executing_file_as_program_appimage.webp)

如果您更倾向于使用命令行，可以打开终端并运行以下命令使 AppImage 可执行：

```bash
sudo chmod a+x ~/Downloads/krita*.appimage
```

## 运行使用 AppImage 的程序

您已经进入了最后一步——运行您的 AppImage！

!!! 注意

    运行 AppImage 不会像传统软件包那样将程序安装到系统文件中。这意味着每当您想使用该程序时，都必须双击 AppImage。因此，将 AppImage 存储在一个安全且容易记住的位置非常重要。

双击 AppImage：

![AppImage 已选中，表示它已被双击并运行](images/run_app_image.webp)

您也可以运行以下 shell 命令来替代双击 AppImage：

 ```bash
    ./krita*.appimage
```

运行 AppImage 后不久，Krita 将会启动。

![Krita 应用启动画面](images/krita_launching.webp)

## 结论

本指南教您如何使用 AppImage 下载和使用程序。AppImage 对最终用户非常方便，因为他们无需了解如何管理仓库、从源码构建或使用命令行，就能使用提供 AppImage 的常用程序。
