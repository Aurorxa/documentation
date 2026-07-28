---
title: 在 Rocky Linux 上设置 libvirt
author: Howard Van Der Wal
contributors: Steven Spencer 
ai_contributors: Gemma 4 (gemma-4-31B-it-UD-Q4_K_XL)
tested with: 9, 10
tags:
- libvirt
- kvm
- virtualization
---

## AI 使用说明

本文档遵循[此处提供的 AI 贡献政策。](../contribute/ai-contribution-policy.md) 如果你在说明中发现任何错误，请告诉我们。

## 简介

libvirt^1^ 是一个功能强大的虚拟化 API，借助 KVM 作为 hypervisor (虚拟机管理器)、QEMU 作为仿真器，可以虚拟化几乎任何你选择的操作系统。

本文档将提供在 Rocky Linux 9 和 10 上设置 libvirt 的说明。两个版本之间步骤不同的地方都有明确标注。
 
## 前提条件

* 一台运行 Rocky Linux 9 或 10 的机器。
* 确保在 BIOS 设置中启用了虚拟化。如果以下命令返回输出，则虚拟化已成功启用：

```bash
sudo grep -e 'vmx' /proc/cpuinfo
```

## 仓库设置与软件包安装

* 启用 EPEL (企业 Linux 额外软件包) 仓库：

```bash
sudo dnf install -y epel-release
```

* 安装 `libvirt` 所需的软件包（可选安装 `virt-manager`，如果你想使用 GUI 管理你的虚拟机）：

```bash
sudo dnf install -y bridge-utils virt-top libguestfs-tools bridge-utils virt-viewer qemu-kvm libvirt virt-manager virt-install
```

!!! note
    `virt-manager` 软件包已被弃用，在 Rocky Linux 10 中不再可用（在 Rocky Linux 9 中仍可用）。对于 Rocky Linux 10，你应安装以下软件包并启用 `cockpit` 服务：

    ```bash
    sudo dnf install -y bridge-utils virt-top libguestfs-tools bridge-utils virt-viewer qemu-kvm libvirt virt-install
    sudo dnf install -y cockpit cockpit-machines
    sudo systemctl enable --now cockpit.socket
    ```

    安装后，你可以通过导航到 `https://<your-host>:9090` 的 Cockpit 控制台来管理你的虚拟机。

## libvirt 用户设置

* 将你的用户添加到 `libvirt` 组。这可以管理你的虚拟机，并允许你以非 root 用户身份使用命令，如 `virt-install`：

```bash
sudo usermod -aG libvirt $USER
```

* 使用 `newgrp` 命令激活 `libvirt` 组：

```bash
newgrp libvirt
```

* 启用并启动 `libvirtd` 服务：

```bash
sudo systemctl enable --now libvirtd
```

## 设置桥接接口以直接访问虚拟机

* 检查当前正在使用的接口，并记下具有 Internet 连接的主接口：

```bash
nmcli connection show
```

* 删除连接到 Internet 的接口以及当前存在的任何虚拟桥接连接：

```bash
nmcli connection delete <CONNECTION_NAME>
```

!!! warning

    确保你对机器有直接访问权限。如果你是通过 SSH 配置机器，删除主接口连接后将断开连接。虽然你可以通过远程系统上的 BASH 脚本运行以下命令（直到 "nmcli connection up"），但这有风险，理想情况下，通过直接或控制台连接拥有另一种连接到系统的方式是最佳方法。

* 创建新的桥接连接：

```bash
nmcli connection add type bridge autoconnect yes con-name <VIRTUAL_BRIDGE_CON-NAME> ifname <VIRTUAL_BRIDGE_IFNAME>
```

* 分配静态 IP 地址：

```bash
sudo nmcli connection modify <VIRTUAL_BRIDGE_CON-NAME> ipv4.addresses <STATIC_IP/SUBNET_MASK> ipv4.method manual
```

* 分配网关地址：

```bash
nmcli connection modify <VIRTUAL_BRIDGE_CON-NAME> ipv4.gateway <GATEWAY_IP>
```

* 分配 DNS 地址：

```bash
nmcli connection modify <VIRTUAL_BRIDGE_CON-NAME> ipv4.dns <DNS_IP>
```

* 添加桥接从属连接：

```bash
nmcli connection add type bridge-slave autoconnect yes con-name <MAIN_INTERFACE_WITH_INTERNET_ACCESS_CON-NAME> ifname <MAIN_INTERFACE_WITH_INTERNET_ACCESS_IFNAME> master <VIRTUAL_BRIDGE_CON-NAME>
```

* 启动桥接连接：

```bash
nmcli connection up <VIRTUAL_BRIDGE_CON-NAME>
```

* 将 `allow all` 行添加到 `bridge.conf`：

```bash
sudo tee -a /etc/qemu-kvm/bridge.conf <<EOF
allow all
EOF
```

* 启用并启动 `libvirtd` 服务：

```bash
sudo systemctl enable --now libvirtd
```

## 在 `/var/lib/libvirt/` 之外安装和运行虚拟机（适用于 Rocky Linux 10）

```bash
# 授予 qemu 用户对存储 ISO 的目录的读取和遍历访问权限
sudo setfacl -R -m u:qemu:rx <ISO_DIRECTORY>
sudo setfacl -d -m u:qemu:rx <ISO_DIRECTORY>

# 配置 SELinux 以允许读取访问
sudo semanage fcontext -a -t virt_image_t "<ISO_DIRECTORY>os(/.*)?"
sudo restorecon -Rv <ISO_DIRECTORY>

# 1. 授予对家目录的遍历访问权限
sudo setfacl -m u:qemu:x <HOME_DIRECTORY>

# 2. 授予对将存储虚拟机的镜像目录的访问权限
sudo setfacl -m u:qemu:rwx <IMAGES_DIRECTORY>

# 3. 确保目录中新创建的文件继承这些权限
sudo setfacl -d -m u:qemu:rwx <IMAGES_DIRECTORY>

# 4. 为目录及其内容定义 SELinux 上下文
sudo semanage fcontext -a -t virt_image_t "<IMAGES_DIRECTORY>(/.*)?"
sudo restorecon -Rv <IMAGES_DIRECTORY>
```

## 虚拟机安装

* 将 `/var/lib/libvirt` 目录及其嵌套目录的所有权设置为你的用户：

```bash
sudo chown -R $USER:libvirt /var/lib/libvirt/
```

* 你可以使用 `virt-install` 命令在命令行创建虚拟机。例如，要创建一个 Rocky Linux 9.8 虚拟机，你可以运行以下命令：

```bash
virt-install --name Rocky-Linux-9 --ram 4096 --vcpus 4 --disk path=/var/lib/libvirt/images/rocky-linux-9.img,size=20 --os-variant rocky9 --network bridge=virbr0,model=virtio --graphics none --console pty,target_type=serial --extra-args 'console=ttyS0,115200n8' --location ~/isos/Rocky-9.8-x86_64-dvd.iso
```

* 对于那些希望通过 GUI 管理虚拟机的人来说，对于 Rocky Linux 9，`virt-manager` 是完美的工具。对于 Rocky Linux 10，在 `virt-manager` 被弃用后，Cockpit 现在是标准。

## 如何关闭虚拟机

* `shutdown` 命令可以完成此操作：

```bash
virsh shutdown --domain <YOUR_VM_NAME>
```

* 要强制关闭无响应的虚拟机，使用 `destroy` 命令：

```bash
virsh destroy --domain <YOUR_VM_NAME>
```

## 如何删除虚拟机

* 使用 `undefine` 命令：

```bash
virsh undefine --domain <YOUR_VM_NAME> --nvram
```

* 有关更多 `virsh` 命令，请查看 `virsh` 的 `man` 手册页。

## 结论

* libvirt 让你能够轻松安装和管理虚拟机，同时还提供广泛的 XML 编辑选项、SecureBoot、cloud-init 支持等。

## 参考文献

1. "libvirt.org" by the libvirt project [https://libvirt.org/index.html](https://libvirt.org/index.html)
