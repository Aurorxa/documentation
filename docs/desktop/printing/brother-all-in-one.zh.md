---
title: Brother 一体机打印机安装与设置
author: Joseph Brinkman
contributors: Steven Spencer, Ganna Zhyrnova
tested with: 9.4
tags:
  - desktop
  - printer support
---

## 简介

得益于第三方 Brother 一体机打印机和扫描仪驱动，在 Linux 上使用 Brother 一体机进行打印和扫描是可行的。

!!! info

    本流程已使用 Brother MFC-J480DW 进行测试。

## 前提条件与假设

- Rocky 9.4 Workstation
- `sudo` 权限
- Brother 一体机打印机和扫描仪

本指南假设打印机可通过 USB 直连或 LAN（局域网）从工作站访问。将打印机连接到 LAN 超出了本文范围。

## 在 GNOME 中添加打印机

1. 打开 ++"Settings"++
2. 在左侧菜单中，点击 ++"Printers"++
3. 注意窗口顶部显示 "Unlock to Change Settings" 的横幅
4. 点击 ++"Unlock"++ 并输入 `sudo` 凭据。
5. 点击 ++"Add"++

点击 ++"Add"++ 后，++"Settings"++ 将开始扫描打印机。如果打印机未出现，但您知道其在局域网上的 IP 地址，可以手动输入 IP 地址。将打印机连接到家庭网络超出了本文范围。

此时将启动一个 Software 窗口，尝试查找并安装打印机驱动。通常情况下，这会失败。您需要前往 Brother 网站安装额外驱动。

## 下载与安装驱动

[Brother 驱动安装脚本安装说明:](https://support.brother.com/g/b/downloadlist.aspx?&c=us&lang=en&prod=mfcj480dw_us_eu_as&os=127){target="_blank"}

1. [下载 Brother MFC-J480DW 打印机驱动 bash 脚本](https://support.brother.com/g/b/downloadtop.aspx?c=us&lang=en&prod=mfcj480dw_us_eu_as){target="_blank"}
2. 打开终端窗口。
3. 进入下载文件的目录，例如 `cd Downloads`
4. 执行以下命令解压下载的文件：

    ```bash
    gunzip linux-brprinter-installer-*.*.*-*.gz
    ```

5. 使用 `su` 命令或 `sudo su` 命令获取超级用户授权。
6. 运行该工具：

    ```bash
    bash linux-brprinter-installer-*.*.*-* Brother machine name
    ```

7. 驱动安装将开始。按照安装屏幕的指示操作。

安装过程可能需要一些时间。请等待直到完成。完成后，可以选择发起测试打印。

## 扫描仪支持

Xsane 是一款提供图形用户界面的扫描工具，使用 appstream 仓库中的软件包，无需额外配置。

```bash
sudo dnf install sane-backends sane-frontends xsane
```

Xsane 的图形界面看起来有些复杂，但简单扫描却很直接。启动 Xsane 后，窗口中有按钮可以 ++"Acquire a preview"++（获取预览）。这将对扫描进行预览。准备好扫描后，在主菜单中点击 ++"Start"++ 按钮。

如需更全面的 Xsane 指南，请阅读这篇 [剑桥大学数学学院文章](https://www.maths.cam.ac.uk/computing/printing/xsane){target="_blank"}。

## 结语

安装必要的 Brother 驱动和 Xsane 后，您现在应该能够在 Brother 一体机上执行打印和扫描操作。
