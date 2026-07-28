---
title: Kickstart 文件与 Rocky Linux
author: Howard Van Der Wal
contributors: Steven Spencer
tested with: 10, 9, 8
tags:
- file
- install
- kickstart
- linux
- rocky 
---

# Kickstart 文件与 Rocky Linux

**知识量**: :star: :star:   
**阅读时间**: 15 分钟

## 简介

Kickstart 文件是同时在一台或多台机器上安装和配置 Rocky Linux 的不可或缺的工具。您可以使用它们来快速设置一切，从您喜爱的游戏工作站到在企业组织中部署数百台机器。它们节省了大量的时间和精力，无需逐一手动配置每台机器。

在本文结束时，您将了解 `kickstart` 文件如何工作，如何创建并应用您自己的 `kickstart` 文件到 Rocky Linux ISO，并能够配置您自己的机器。

## 什么是 kickstart 配置？

Kickstart 文件是一组配置文件的集合，用户可以使用它们快速、轻松地部署 Linux 发行版。Kickstart 文件不仅适用于 Rocky Linux，还适用于 CentOS Stream、Fedora 以及许多其他发行版。

## Kickstart 配置如何应用到一个 ISO？

使用 `mkkiso`，一个 kickstart 文件被复制到 Linux ISO 的 `root` 目录中。然后 `mkkiso` 编辑 `/isolinux/isolinux.cfg`，并将 `kickstart` 文件作为内核命令行参数（`inst.ks`）放在其中：

```bash
sudo mount -o ro,loop rocky-linux10-dvd-shadow.iso /mnt/
cat /mnt/EFI/BOOT/grub.cfg | grep shadow | head -1
linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:LABEL=Rocky-10-1-x86_64-dvd quiet inst.ks=hd:LABEL=Rocky-10-1-x86_64-dvd:/rocky_linux10_shadow.ks
```

完成后，`mkkiso` 会生成一个内置了 `kickstart` 配置的新 ISO。当 ISO 启动时，`anaconda` 将运行 `kickstart` 文件中列出的指令。

## 前置条件

* 可选 - 通过 PXE 服务器部署您的 `kickstart` ISO：查看 [如何在 Rocky Linux 9.x 上设置 PXE 服务器](https://kb.ciq.com/article/rocky-linux/rl-pxe-boot-kickstart-file) 指南了解更多。

* 一根 USB 3.0+ 存储棒用于 USB 安装。

* 从 https://rockylinux.org/download 下载 Rocky Linux 8、9 或 10 Minimal ISO（DVD ISO 并非必需）。

* 按照 [自定义 Rocky Linux ISO 设置步骤](https://docs.rockylinux.org/10/guides/isos/iso_creation/) 指南安装 `lorax` 包并学习如何生成 Rocky Linux `kickstart` ISO。

## Kickstart 示例

=== "10"


    ```
    lang en_GB
    keyboard --xlayouts='jp'
    timezone Asia/Tokyo --utc
    rootpw --iscrypted $6$0oXug1vTr7TO3kJu$/kvm.lctWsLDHaeak/YuUaEu26LzvNuE1L/tvUC4G91ZroChjDTUDwQDEkQfGhwQw4doiDcZc2P6et.zzRqOZ/ --allow-ssh
    user --name howard --password $6$8wzUW5ipTdTs.MbM$1F6mPfqQAXPeSVArqT2r/GL6QljXs2dQWCcNGjQq5cpEPGWhNvOCAiVCDJRA0FZQpoTXJSBtNON2ZqvEMBUNX/ --iscrypted --groups=wheel
    reboot
    text
    url --url='https://download.rockylinux.org/pub/rocky/10/BaseOS/x86_64/os/'
    bootloader --location=boot --append="ro crashkernel=2G-64G:256M,64G-:512M rhgb quiet"
    zerombr
    clearpart --all --initlabel --disklabel=gpt
    ignoredisk --only-use=nvme0n1
    part /boot/efi --fstype=efi --size=600
    part /boot --fstype=xfs --size=1024
    part pv.0 --fstype=lvmpv --size=480000
    volgroup rl --pesize=4096 pv.0
    logvol / --vgname=rl --name=root --fstype=xfs --size=70000
    logvol swap --vgname=rl --name=swap --fstype=swap --size=1024
    logvol /home --vgname=rl --name=home --fstype=xfs --size=1000 --grow
    network --device=enp4s0 --hostname=shadow --bootproto=static --ip=192.168.1.102 --netmask=255.255.255.0 --gateway=192.168.1.1 --nameserver=192.168.1.1 --activate
    skipx
    firstboot --disable
    selinux --enforcing
    firewall --enabled --ssh
    %packages
    @^server-product-environment
    %end

    %post
    mkdir -p /mnt/storage1
    mkdir -p /mnt/storage2
    mkfs.xfs /dev/nvme0n1
    mkfs.xfs /dev/sda
    sync
    udevadm settle
    sleep 2
    UUID_NVME0N1=$(blkid -s UUID -o value /dev/nvme0n1)
    UUID_SDA=$(blkid -s UUID -o value /dev/sda)
    if [ -n "$UUID_NVME0N1" ]; then
        echo "UUID=$UUID_NVME0N1 /mnt/storage1 xfs defaults,inode64 0 0" >> /etc/fstab
    fi
    if [ -n "$UUID_SDA" ]; then
        echo "UUID=$UUID_SDA /mnt/storage2 xfs defaults,inode64 0 0" >> /etc/fstab
    fi
    mount -U $UUID_NVME0N1 /mnt/storage1
    mount -U $UUID_SDA /mnt/storage2
    chown -R howard:howard /mnt/storage1
    chown -R howard:howard /mnt/storage2
    %end
    ```


=== "9"


    ```
    lang en_GB
    keyboard --xlayouts='jp'
    timezone Asia/Tokyo --utc
    rootpw --iscrypted $6$IjIs0nEufTOaj2cZ$EnZKdrjHQ9OmhePUMVWUcaJmmC0vU2L.b02lBMKmRMLq/VZOhnrgBO64ru29rFnB8HQsGo0cQLqBoLIpL7PbS1 --allow-ssh
    user --name howard --groups wheel --password $6$OdZuQb9owvkol5gv$6X7w0VraE7hDSrrHS5oz9BvNACB.PcrNt5Ulka9/g1Sgxdzl93LAuGT3GH8a.4ZUpqzchKU3glgRyCWXhSN68. --iscrypted
    reboot
    text
    url --url='https://download.rockylinux.org/pub/rocky/9/BaseOS/x86_64/os/'
    bootloader --location=boot --append="crashkernel=1G-4G:192M,4G-64G:256M,64G:512M rhgb quiet"
    zerombr
    clearpart --all --initlabel --disklabel=gpt
    ignoredisk --only-use=sda
    part /boot/efi --fstype=efi --size=600
    part /boot --fstype=xfs --size=1024
    part pv.0 --fstype=lvmpv --size=120012
    volgroup rl --pesize=4096 pv.0
    logvol / --vgname=rl --name=root --fstype=xfs --size=70000
    logvol swap --vgname=rl --name=swap --fstype=swap --size=1024
    logvol /home --vgname=rl --name=home --fstype=xfs --size=1000 --grow
    network --device=enp2s0 --hostname=mighty --bootproto=static --ip=192.168.1.104 --netmask=255.255.255.0 --gateway=192.168.1.1 --nameserver=192.168.1.1 --activate
    skipx
    firstboot --disable
    selinux --enforcing
    firewall --enabled --ssh
    %packages
    @^server-product-environment
    %end
    ```


=== "8"


    ```
    lang en_GB
    keyboard jp106
    timezone Asia/Tokyo --utc
    rootpw <ROOT_PASSWORD_HERE> --iscrypted
    user --name=howard --password=<USER_PASSWORD_HERE> --iscrypted --groups=wheel
    reboot
    text
    url --url='https://download.rockylinux.org/pub/rocky/8/BaseOS/x86_64/os/'
    bootloader --append="rhgb quiet crashkernel=auto"
    zerombr
    clearpart --all --initlabel
    autopart
    network --device=enp1s0 --hostname=rocky-linux8-slurm-controller-node --bootproto=static --ip=192.168.1.120 --netmask=255.255.255.0 --gateway=192.168.1.1 --nameserver=192.168.1.1
    firstboot --disable
    selinux --enforcing
    firewall --enabled --ssh
    %packages
    @^server-product-environment
    %end 
    ```


**下面重点突出了一些感兴趣的项目，重点放在 Rocky Linux 10 的 `kickstart` 文件上。同时讨论了不同 `kickstart` 文件之间的差异：**

### Rocky Linux 10 kickstart 文件分解说明

#### rootpw

!!! warning

    始终对 root 密码使用 `--iscrypted` 选项，以确保其不以明文形式显示。

要生成所需密码的哈希值，使用以下 `openssl` 命令并在提示时输入密码：

```bash
openssl passwd -6
```

要允许通过 `ssh` 访问 `root` 账户，在 `rootpw` 行中添加 `--allow-ssh` 选项。

#### user 

同样地，使用 `--iscrypted` 选项以确保您的密码不以明文形式显示。

如果您想将用户设为管理员，使用 `--groups=wheel` 将其添加到 `wheel` 组。

#### URL

将 `cdrom` 选项与 `ignoredisk` 搭配使用会导致问题：Anaconda 无法访问 USB 驱动器并且在存储配置期间挂起。使用 `url --url` 通过从 `BaseOS` 下载安装来解决该问题。

#### Bootloader

设置 bootloader 的位置并附加所需的内核命令行参数。

#### Zerombr

确保销毁 Anaconda 在所选磁盘上无法识别的任何分区表或其他格式化选项。

#### Clearpart

擦除目标磁盘上的所有分区，并将磁盘标签设置为 `gpt`。

#### Ignoredisk

如果未指定 `ignoredisk`，`anaconda` 将可以访问系统上的所有磁盘。如果指定了，`anaconda` 将只使用用户选择的磁盘。

#### Part

`part` 允许用户指定他们想要创建的分区。上述示例展示了一个 `/boot`、`/boot/efi` 和 LVM（逻辑卷管理）配置。这与执行 Rocky Linux 自动安装时得到的结果相同。

#### Volgroup

`volgroup` 创建 LVM 组。此示例显示选择名称为 `rl`，物理扩展（`pesize`）为 `4096 KiB`。

#### Logvol

在 LVM 组下创建逻辑卷。注意 `/home` 卷的 `--grow` 选项，确保使用了 LVM 组的全部空间。

#### Network

此处可以选择静态或动态设置网络配置。

#### Skipx

停止系统上的 X 服务器配置。

#### Firstboot

在本例中，我们设置了 `--disable` 标志，这会在系统启动时阻止 Setup Agent 启动。

#### Firewall

通过防火墙允许 `ssh` 访问（使用 `--ssh`）非常重要，这样在没有控制台访问权限时也能登录机器。

#### %packages

列出要安装的软件包。在示例中，`@^server-product-environment` 软件包组是安装的候选目标。这将安装稳定 Rocky Linux 服务器所需的所有必要软件包。

此外，您也可以在此处选择要安装的单个软件包，排除某些软件包的安装等。

#### %post

在操作系统安装完成后，您也可以在此列出需要执行的额外任务。在给出的示例中，作者正在配置并挂载系统中可用的额外存储。

此处还提供其他选项，如 `%pre`、`%pre-install`、`%onerror` 和 `%traceback`。您可以通过本文档末尾提供的参考资料了解更多关于这些选项的信息。

### Rocky Linux kickstart 之间的显著差异

Rocky Linux 10 和 9 通过以下风格定义键盘布局（以日语键盘为例）：

```
keyboard --xlayouts='jp'
```

而 Rocky Linux 8 将键盘布局定义为：

```
keyboard jp106
```

对于通过 `ssh` 访问 `root` 账户，Rocky Linux 8 的 `kickstart` 文件**不**需要添加 `--allow-ssh` 标志。

`crashkernel` 内核命令行参数在所有三个 Rocky Linux 版本之间有所不同。在设置该参数时请注意这一点。

在 Rocky Linux 8 的 `kickstart` 示例中（适用于所有 Rocky Linux 版本），如果您希望自动分区驱动器，只需设置 `autopart` 选项。

## 总结

如果您想自动化 Rocky Linux 安装，那么 `kickstart` 文件就是正确的方式。本指南只是您可以使用 `kickstart` 文件所能完成工作的冰山一角。如需了解所有可用 `kickstart` 选项及示例，请查看 Chris Lumens 和 Anaconda 安装团队编写的 `kickstart` 文档^2^。

对于那些希望在虚拟机部署领域进一步推动自动化并同时利用其 `kickstart` 知识的人，Antoine Le Morvan 编写了一份优秀的指南^1^，介绍如何使用 `packer` 实现这一目标。

Rocky Linux 发布工程团队在 Rocky Linux 仓库^4^中也提供了多个 `kickstart` 文件示例。

最后，如果您有 Red Hat 账户，Red Hat 提供了一个 Kickstart Generator，允许您通过 UI 快速便捷地创建 `kickstart` 文件。

## 参考资料

1. "Automatic template creation with Packer and deployment with Ansible in a VMware vSphere environment" by Antoine Le Morvan [https://docs.rockylinux.org/10/guides/automation/templates-automation-packer-vsphere/](https://docs.rockylinux.org/10/guides/automation/templates-automation-packer-vsphere/) 
2. "Extensive kickstart documentation" by Chris Lumens and the Anaconda installer team [https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html)
3. "Red Hat Kickstart Generator (requires a Red Hat Account)" by Red Hat [https://access.redhat.com/labsinfo/kickstartconfig](https://access.redhat.com/labsinfo/kickstartconfig)
4. "Rocky Linux Kickstart Repository" by the Rocky Linux Release Engineering Team [https://github.com/rocky-linux/kickstarts/tree/main](https://github.com/rocky-linux/kickstarts/tree/main)
Footer
