---
title: 安装 NVIDIA GPU 驱动
author: Joseph Brinkman
contributors: Steven Spencer, Ganna Zhyrnova
---

## 介绍

NVIDIA^&reg;^ 是最受欢迎的 GPU（图形处理单元）制造商之一。您可以通过多种方式安装 NVIDIA GPU 驱动。本指南使用 NVIDIA 官方仓库来安装其驱动。因此，此处大量引用了 [NVIDIA 驱动安装指南](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/index.html)。

安装 NVIDIA 驱动的其他替代方法包括：

* NVIDIA 的 `.run` 安装程序
* 第三方 RPMFusion 仓库
* 第三方 ELRepo 驱动

在大多数情况下，最好从官方来源安装 NVIDIA 驱动。RPMFusion 和 ELRepo 可供那些偏好社区仓库的用户使用。对于较旧的硬件，RPMFusion 效果最佳。建议避免使用 `.run` 安装程序。虽然方便，但使用 `.run` 安装程序因覆盖系统文件并出现兼容性问题而臭名昭著。

NVIDIA Linux GPU 驱动包含两种"口味"的内核模块：支持 Turing 及更新架构的开放内核模块（open kernel module）风格，以及旧架构所需的专有内核模块（proprietary kernel module）风格。请注意，自 560 驱动版本以来，开放内核模块风格已成为默认和推荐的安装方式，而更新的 GPU 系列如 RTX 50 系列仅支持开放内核模块风格。

## 假设

对于本指南，您需要以下条件：

* Rocky Linux Workstation
* `sudo` 权限

## 安装必要的工具和依赖项

启用 EPEL（Extra Packages for Enterprise Linux）仓库：

```bash
sudo dnf install epel-release -y
```

启用 CRB（CodeReady Builder）仓库：

```bash
sudo dnf config-manager --enable crb
```

安装开发工具以确保必要的构建依赖项：

```bash
sudo dnf groupinstall "Development Tools" -y
```

`kernel-devel` 包提供构建内核模块所需的头文件和工具：

```bash
sudo dnf install kernel-devel-matched kernel-headers -y
```

## 安装 NVIDIA 驱动

安装必要的先决条件后，是时候安装 NVIDIA 驱动了。

使用以下命令添加官方 NVIDIA 仓库：

```bash
sudo dnf config-manager --add-repo http://developer.download.nvidia.com/compute/cuda/repos/rhel10/$(uname -m)/cuda-rhel10.repo
```

接下来，清理 DNF 仓库缓存：

```bash
sudo dnf clean expire-cache
```

最后，为您的系统安装最新的 NVIDIA 驱动。对于开放内核模块，运行：

```bash
sudo dnf install nvidia-open -y
```

对于专有内核模块，运行：

```bash
sudo dnf install cuda-drivers -y
```

### 较旧 GPU
NVIDIA 驱动的 590 版本[放弃了对基于 Maxwell、Pascal 和 Volta 架构的 GPU 的支持](https://forums.developer.nvidia.com/t/unix-graphics-feature-deprecation-schedule/60588)。在此类系统上，上述指令将安装驱动且无错误，但重启后，由于找不到任何支持的 GPU，将无法加载模块。但是，如果您拥有此类 GPU，仍可以安装较旧的驱动：

```bash
sudo dnf install cuda-drivers-580 -y
```

然后，您需要通过 [dnf 的 versionlock 插件](https://docs.rockylinux.org/books/admin_guide/13-softwares/#versionlock-plugin)保护 `cuda-drivers` 包免受未来更新的影响。


## 禁用 Nouveau

Nouveau 是一个开源的 NVIDIA 驱动，与 NVIDIA 的专有驱动相比，其功能有限。最好禁用它以避免驱动冲突：

```bash
sudo grubby --args="nouveau.modeset=0 rd.driver.blacklist=nouveau" --update-kernel=ALL
```

!!! Note

    对于启用了 Secure Boot 的系统，您需要执行此步骤：

    ```bash
    sudo mokutil --import /var/lib/dkms/mok.pub
    ```

    `mokutil` 命令将提示您创建一个密码，该密码将在重启期间使用。
    
    重启后，系统应询问您是否要 enroll a key 或类似的内容，选择 "yes"，然后它会要求您提供在 `mokutil` 命令中输入的密码。

重启：

```bash
sudo reboot now
```

## 总结

您已成功使用 NVIDIA 官方仓库在系统上安装了 NVIDIA GPU 驱动。享受默认 Nouveau 驱动无法提供的 NVIDIA GPU 增强功能。
