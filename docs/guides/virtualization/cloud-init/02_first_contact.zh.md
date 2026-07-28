---
title: 2. 初次接触
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - cloud-init
  - cloud
  - automation
---

## 使用 Rocky Linux 10 QCOW2 镜像进行简单引导启动

在前一章中，我们介绍了 `cloud-init` 的基本概念。现在是从理论走向实践的时候了。本章是你的第一个动手任务：你将获取官方的 Rocky Linux 10 通用云镜像，为其提供一组简单的指令，并观察它在首次启动时自行配置。

## 1. 准备实验环境

在启动第一个实例之前，我们需要准备好本地实验环境。对于这些练习，我们将使用标准的 Linux 虚拟化工具来模拟云环境。

### 前提条件：宿主机工具

确保你的宿主机上安装了以下工具。在 Rocky Linux 宿主机上，可以使用 `dnf` 安装：

```bash
sudo dnf install -y libvirt qemu-kvm virt-install genisoimage
```

* **虚拟化 Hypervisor：** 像 KVM/QEMU 或 VirtualBox 这样的工具。
* `virt-install`：用于配置新虚拟机的命令行工具。
* `genisoimage`（或 `mkisofs`）：用于创建 ISO9660 文件系统的工具。

### QCOW2 镜像

如果你还没有下载，可以使用以下命令下载官方的 Rocky Linux 10 通用云镜像：

```bash
curl -L -o Rocky-10-GenericCloud.qcow2 \
https://dl.rockylinux.org/pub/rocky/10/images/x86_64/Rocky-10-GenericCloud-Base.latest.x86_64.qcow2
```

为保留原始镜像，请为你的虚拟机创建一个镜像的工作副本。

```bash
cp Rocky-10-GenericCloud.qcow2 first-boot-vm.qcow2
```

!!! tip "使用后备文件节省磁盘空间"

    完整拷贝镜像可能会占用大量空间。为了节省磁盘空间，你可以创建一个 *链接克隆* (linked clone)，使用原始镜像作为后备文件。这会创建一个更小的 `qcow2` 文件，只存储与原始镜像之间的差异。

    ```bash
    qemu-img create -f qcow2 -F qcow2 \
    -b Rocky-10-GenericCloud.qcow2 first-boot-vm.qcow2
    ```

## 2. 方法一：`NoCloud` 数据源（ISO）

在本地环境中向 `cloud-init` 提供数据的最常见方式之一是 `NoCloud` 数据源。此方法要求将配置文件打包到一个虚拟 CD-ROM（ISO 文件）中，`cloud-init` 会在启动时自动检测并读取。

### 创建配置文件

1. **为配置文件创建一个目录：**

    ```bash
    mkdir cloud-init-data
    ```

2. **创建 `user-data` 文件：** 此文件是你的主要指令集。我们将使用 `cat` 的 heredoc 语法来创建它。

    ```bash
    cat <<EOF > cloud-init-data/user-data
    #cloud-config
    hostname: cloud-rockstar-01
    runcmd:
      - [ sh, -c, "echo 'Hello from the cloud-init Final Stage!' > /root/boot_done.txt" ]
    EOF
    ```

3. **创建 `meta-data` 文件：** 此文件提供 *关于* 实例的上下文。`instance-id` 尤为重要，因为 `cloud-init` 用它来判断是否在此实例上运行过。更改 ID 会导致 `cloud-init` 重新运行。

    ```bash
    cat <<EOF > cloud-init-data/meta-data
    {
      "instance-id": "i-first-boot-01",
      "local-hostname": "cloud-rockstar-01"
    }
    EOF
    ```

4. **生成 ISO：** 使用 `genisoimage` 将文件打包成 `config.iso`。卷标 `-V cidata` 是 `cloud-init` 查找的魔术钥匙。

    ```bash
    genisoimage -o config.iso -V cidata -r -J cloud-init-data
    ```

### 启动与验证

1. **使用 `virt-install` 启动虚拟机**，同时挂载虚拟机镜像和 `config.iso`。

    ```bash
    virt-install --name rocky10-iso-boot \
    --memory 2048 --vcpus 2 \
    --disk path=first-boot-vm.qcow2,format=qcow2 \
    --disk path=config.iso,device=cdrom \
    --os-variant rockylinux10 \
    --import --noautoconsole
    ```

2. **找到 IP 地址并通过 SSH 连接。** 默认用户是 `rocky`。

    ```bash
    virsh domifaddr rocky10-iso-boot
    ssh rocky@<IP_ADDRESS>
    ```

    !!! tip "使用 SSH 密钥进行安全登录"

        使用默认用户连接对于快速实验测试很方便，但这并不是一种安全的做法。在下一章中，我们将探讨如何使用 `cloud-init` 自动注入你的 SSH 公钥，从而支持安全、无密码的登录。

3. **验证更改：** 检查主机名以及由 `runcmd` 创建的文件。

    ```bash
    hostname
    sudo cat /root/boot_done.txt
    ```

## 3. 方法二：使用 `virt-install` 直接注入

创建 ISO 是一种可靠的方法，但对于 `libvirt` 和 `virt-install` 的用户，有一种更简单的方式。`--cloud-init` 标志允许你直接传递 `user-data`，让 `virt-install` 自动处理数据源的创建。

### 简化的 `user-data`

创建一个简单的 `user-data.yml` 文件。你甚至可以提前添加 SSH 密钥。

```bash
cat <<EOF > user-data.yml
#cloud-config
users:
  - name: rocky
    ssh_authorized_keys:
      - <YOUR_SSH_PUBLIC_KEY>
EOF
```

### 启动与验证

1. **使用 `--cloud-init` 标志启动虚拟机。** 注意，我们可以直接在此设置主机名。

    ```bash
    virt-install --name rocky10-direct-boot \
    --memory 2048 --vcpus 2 \
    --disk path=first-boot-vm.qcow2,format=qcow2 \
    --cloud-init user-data=user-data.yml,hostname=cloud-rockstar-02 \
    --os-variant rockylinux10 \
    --import --noautoconsole
    ```

2. **找到 IP 地址并连接。** 如果你添加了 SSH 密钥，你应该可以无密码连接。

3. **验证主机名。** 它应该是 `cloud-rockstar-02`。

这种直接注入方法对于使用 `libvirt` 进行本地开发和测试通常更快、更方便。
