---
title: 将 Rocky Linux 导入 WSL 或 WSL2
author: Lukas Magauer
tested_with: 8.10, 9.6, 10.0
tags:
  - wsl
  - wsl2
  - windows
  - interoperability
---

!!! note "其他版本的镜像"

    如果您在寻找其他版本 Rocky Linux 的 WSL 操作说明，请从顶部菜单选择所需版本，然后参考"互操作性"下的 WSL 操作说明。

## 前置条件

您必须启用 Windows-Subsystem for Linux 功能。通过以下选项之一完成：

- [Microsoft Store 中提供了具有额外功能的较新 WSL 版本](https://apps.microsoft.com/store/detail/windows-subsystem-for-linux/9P9TQF7MRM4R)。请尽可能使用这个较新版本。
- 打开管理终端（PowerShell 或命令提示符），运行 `wsl --install`（[参考](https://docs.microsoft.com/en-us/windows/wsl/install)）
- 进入图形化 Windows 设置，启用可选功能 `Windows-Subsystem for Linux`

此功能应在当前所有受支持的 Windows 10 和 11 版本上可用。

!!! tip "WSL 版本"

    请确保您的 WSL 版本是最新的，因为某些功能仅在后续版本中引入。如果不确定，请运行 `wsl --update`。

## 步骤

### 可安装的 WSL 镜像（推荐）

1. 从 CDN 或离您更近的镜像站下载 WSL 镜像：

    - 10：[x86_64](https://dl.rockylinux.org/pub/rocky/10/images/x86_64/Rocky-10-WSL-Base.latest.x86_64.wsl) 或 [aarch64](https://dl.rockylinux.org/pub/rocky/10/images/aarch64/Rocky-10-WSL-Base.latest.aarch64.wsl)

2. 安装 `.wsl` 镜像有多种选项：

    - 双击镜像，它将使用镜像的默认名称进行安装
    - 通过命令行安装镜像：

        ```sh
        wsl --install --from-file <path-to/Rocky-10-WSL-Base.latest.x86_64.wsl> --name <machine-name>
        ```

### 传统容器镜像

1. 获取容器 rootfs。有多种方式：

    - 从 CDN 下载镜像：
        - 10：[Base x86_64](https://dl.rockylinux.org/pub/rocky/10/images/x86_64/Rocky-10-Container-Base.latest.x86_64.tar.xz)、[Minimal x86_64](https://dl.rockylinux.org/pub/rocky/10/images/x86_64/Rocky-10-Container-Minimal.latest.x86_64.tar.xz)、[UBI x86_64](https://dl.rockylinux.org/pub/rocky/10/images/x86_64/Rocky-10-Container-UBI.latest.x86_64.tar.xz)、[Base aarch64](https://dl.rockylinux.org/pub/rocky/10/images/aarch64/Rocky-10-Container-Base.latest.aarch64.tar.xz)、[Minimal aarch64](https://dl.rockylinux.org/pub/rocky/10/images/aarch64/Rocky-10-Container-Minimal.latest.aarch64.tar.xz)、[UBI aarch64](https://dl.rockylinux.org/pub/rocky/10/images/aarch64/Rocky-10-Container-UBI.latest.aarch64.tar.xz)
    - 从 Docker Hub 或 Quay.io 导出镜像（[参考](https://docs.microsoft.com/en-us/windows/wsl/use-custom-distro#export-the-tar-from-a-container)）

        ```sh
        <podman/docker> export rockylinux:10 > rocky-10-image.tar
        ```

2. （可选）如果使用的是较新版本的 WSL，需要从 .tar.xz 文件中提取 .tar 文件
3. 创建 WSL 存储文件的目录（通常在用户配置文件的某个位置）
4. 最后，将镜像导入 WSL（[参考](https://docs.microsoft.com/en-us/windows/wsl/use-custom-distro#import-the-tar-file-into-wsl)）：

    - WSL：

        ```sh
        wsl --import <machine-name> <path-to-vm-dir> <path-to/rocky-10-image.tar.xz> --version 1
        ```

    - WSL 2：

        ```sh
        wsl --import <machine-name> <path-to-vm-dir> <path-to/rocky-10-image.tar.xz> --version 2
        ```

!!! tip "WSL 与 WSL 2"

    一般来说，WSL 2 应比 WSL 更快，但具体取决于使用场景。

!!! tip "Windows Terminal"

    如果您安装了 Windows Terminal，新的 WSL 发行版名称将作为下拉菜单中的一个选项出现，这样以后启动非常方便。您还可以使用颜色、字体和其他元素进行自定义。

!!! tip "systemd"

    WSL 镜像默认已启用 systemd。如果要使用容器镜像或自行构建，只需在 `/etc/wsl.conf` 文件的 `boot` 部分添加 `systemd=true` 即可。（[参考](https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/#set-the-systemd-flag-set-in-your-wsl-distro-settings)）
