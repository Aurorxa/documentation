---
title: 5. 镜像构建者的视角
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - cloud-init
  - rocky linux
  - cloud
  - automation
  - image-building
---

## 默认值与通用化 (Generalization)

到目前为止，我们的旅程一直聚焦于在启动时使用 `user-data` 配置单个实例。在本章中，我们将视角转变为 **镜像构建者** (image builder) 的视角。也就是说，那些创建和维护"黄金镜像" (golden images)、作为其他虚拟机模板的人。

我们的目标是创建一个新的自定义镜像，其中包含我们自己预制的策略和默认值。这涉及两个关键过程：

1. **自定义系统范围的默认值：** 修改 *镜像内部的* `cloud-init` 配置。
2. **通用化镜像：** 使用诸如 `cloud-init clean` 和 `virt-sysprep` 等工具剥离所有机器特定数据，为镜像克隆做好准备。

## 1. 自定义实验环境设置

首先，我们需要一个可以修改的基础云镜像的运行实例。我们将在 *不* 提供任何 `user-data` 的情况下启动此虚拟机，以获得一个干净的系统。

```bash
# 为我们的新模板创建一个磁盘镜像
qemu-img create -f qcow2 -o size=10G golden-image-template.qcow2

# 使用 virt-install 启动基础镜像
virt-install --name golden-image-builder \
--memory 2048 --vcpus 2 \
--disk path=golden-image-template.qcow2,format=qcow2 \
--cloud-init none --os-variant rockylinux10 --import

# 连接到控制台并以默认的 'rocky' 用户身份登录
virsh console golden-image-builder
```

## 2. 使用 `cloud.cfg.d` 进行系统范围配置

在我们运行的虚拟机内部，现在可以自定义系统范围的 `cloud-init` 配置了。你绝对不应该直接编辑主配置文件 `/etc/cloud/cloud.cfg`。进行自定义的正确、升级安全的位置是 `/etc/cloud/cloud.cfg.d/` 目录。`cloud-init` 在主 `cloud.cfg` 之后按字母顺序读取此处的所有 `.cfg` 文件，允许你覆盖默认值。

### 动手实践：设置黄金镜像默认值

让我们在黄金镜像上强制执行一个策略：我们将禁用密码认证，设置一个新的默认用户，并确保始终安装一组基准软件包。

1. **创建自定义配置文件：** 在虚拟机内部，创建 `/etc/cloud/cloud.cfg.d/99-custom-defaults.cfg`。`99-` 前缀确保它被最后读取。

    ```bash
    sudo cat <<EOF > /etc/cloud/cloud.cfg.d/99-custom-defaults.cfg
    # Golden Image Customizations

    # 定义一个名为 'admin' 的新默认用户
    system_info:
      default_user:
        name: admin
        sudo: ["ALL=(ALL) NOPASSWD:ALL"]
        shell: /bin/bash

    # 强制使用基于密钥的 SSH 认证
    ssh_pwauth: false

    # 确保始终安装一组基准软件包
    packages:
      - htop
      - vim-enhanced
    EOF
    ```

!!! tip "禁用特定模块"

    一种强大的安全技术是完全禁用特定的 `cloud-init` 模块。例如，要防止任何用户使用 `runcmd`，你可以在自定义 `.cfg` 文件中添加以下内容。这告诉 `cloud-init` 在最终阶段运行一个空的模块列表。

    ```yaml
    cloud_final_modules: []
    ```

## 3. 通用化镜像

我们的虚拟机现在包含我们的自定义配置，以及唯一的机器标识符（如 `/etc/machine-id`）和 SSH 主机密钥。在克隆之前，我们必须在一个称为 **通用化** (generalization) 的过程中移除这些数据。

### 方法一：`cloud-init clean`（在虚拟机内部）

`cloud-init` 提供了一个内置命令来用于此目的。

1. **运行 `cloud-init clean`：** 在虚拟机内部，运行以下命令来剥离实例特定数据。

    ```bash
    sudo cloud-init clean --logs --seed
    ```

    !!! note "关于 `cloud-init clean --seed`"

        此命令移除 `cloud-init` 用于标识实例的唯一种子，强制它在下次启动时从头运行。它 **不会** 移除你在 `/etc/cloud/cloud.cfg.d/` 中的自定义配置。此步骤对于创建真正通用的模板至关重要。

2. **立即关机：** 清理后，立即关闭虚拟机电源。

    ```bash
    sudo poweroff
    ```

### 方法二：`virt-sysprep`（从宿主机）

一个更彻底的、行业标准的工具是 `virt-sysprep`。你可以从宿主机上对已关闭的虚拟机磁盘运行它。它执行 `cloud-init clean` 的所有操作以及更多，例如清除命令历史记录、移除临时文件以及重置日志文件。

1. **确保虚拟机已关闭。**

2. **从你的宿主机运行 `virt-sysprep`：**

    ```bash
    sudo virt-sysprep -a golden-image-template.qcow2
    ```

一旦通用化过程完成，磁盘文件 (`golden-image-template.qcow2`) 就是你的新黄金镜像。

!!! note "黄金镜像命名规范"

    一个好的做法是给黄金镜像起一个描述性的名称，包括操作系统和版本号，例如 `rocky10-base-v1.0.qcow2`。这有助于版本控制和基础设施管理。

## 4. 验证黄金镜像

让我们通过从它启动一个新实例（不带任何 `user-data`）来测试我们的新镜像。

1. **从我们的黄金镜像创建一个新的虚拟机磁盘：**

    ```bash
    qemu-img create -f qcow2 -F qcow2 -b golden-image-template.qcow2 test-instance.qcow2
    ```

2. **启动测试实例：**

    ```bash
    virt-install --name golden-image-test --cloud-init none ...
    ```

3. **验证：** 连接到控制台 (`virsh console golden-image-test`)。登录提示符应该是用户 `admin`，而不是 `rocky`。登录后，你还可以使用 (`rpm -q htop`) 验证 `htop` 的安装。这确认了你的预制默认值正在工作。

## 下一步

你现在已经学会了如何通过预制 `cloud-init` 的系统范围配置默认值来创建标准化模板，并正确地将它们通用化以便克隆。在下一章中，我们将介绍当 `cloud-init` 行为不符合预期时进行故障排除的关键技能。
