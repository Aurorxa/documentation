---
title: 4. 高级配置
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - cloud-init
  - rocky linux
  - cloud
  - automation
  - networking
---

## 网络与多部分负载 (Multi-Part Payloads)

在上一章中，你掌握了用于管理用户、软件包和文件的核心 `cloud-init` 模块。现在你可以以声明式的方式构建一台配置完善的服务器。现在是探索更高级技术的时候了，这些技术能让你对实例配置拥有更大的控制权。

本章涵盖两个强大的高级主题：

1. 声明式网络配置：如何超越 DHCP (动态主机配置协议)，为你的实例定义静态网络配置。
2. 多部分 MIME (多用途互联网邮件扩展) 负载：如何将不同类型的 user-data（如 Shell 脚本和 `#cloud-config` 文件）组合成一个强大的负载。

## 1. 声明式网络配置

默认情况下，大多数云镜像的配置是通过 DHCP 获取 IP 地址。虽然这很方便，但许多生产环境要求服务器具有可预测的静态 IP 地址。`cloud-init` 网络配置系统提供了一种平台无关的、声明式的方式来管理这一点。

网络配置在与主 `#cloud-config` 文件分开的 YAML 文档中指定。`cloud-init` 从同一个文件中处理两者，使用标准的 YAML 文档分隔符（`---`）来区分它们。

!!! note "`cloud-init` 如何应用网络状态"

    在 Rocky Linux 上，`cloud-init` 不直接配置网络接口。相反，它充当翻译器，将其网络配置转换为 **NetworkManager**（默认网络服务）能理解的文件。然后它将控制权交给 NetworkManager 来应用配置。你可以检查 `/etc/NetworkManager/system-connections/` 中的结果连接配置文件。

### 示例 1：配置单个静态 IP

在此练习中，我们将为虚拟机配置一个静态 IP 地址、一个默认网关和自定义 DNS 服务器。

1. **创建 `user-data.yml`：**

    此文件包含两个不同的 YAML 文档，由 `---` 分隔。第一个是我们的标准 `#cloud-config`。第二个定义了网络状态。

    ```bash
    cat <<EOF > user-data.yml
    #cloud-config
    # 我们仍然可以包含标准模块。
    # 让我们安装一个网络故障排除工具。
    packages:
      - traceroute

    ---

    # 第二个文档定义网络配置。
    network:
      version: 2
      ethernets:
        eth0:
          dhcp4: no
          addresses:
            - 192.168.122.100/24
          gateway4: 192.168.122.1
          nameservers:
            addresses: [8.8.8.8, 8.8.4.4]
    EOF
    ```

2. **关键指令解释：**

    * `network:`：网络配置的顶级键。
    * `version: 2`：指定我们使用的是现代的、类似 Netplan 的语法。
    * `ethernets:`：要配置的物理网络接口字典，以接口名称（例如 `eth0`）为键。
    * `dhcp4: no`：在此接口上禁用 IPv4 DHCP。
    * `addresses`：以 CIDR 表示法指定的要分配的静态 IP 地址列表。
    * `gateway4`：IPv4 流量的默认网关。
    * `nameservers`：包含用于 DNS 解析的 IP 地址列表的字典。

3. **启动并验证：**

    这次的验证方式不同，因为虚拟机不会获取动态 IP 地址。你必须直接连接到虚拟机的控制台。

    ```bash
    # 为本练习使用一个新的磁盘镜像
    qemu-img create -f qcow2 -F qcow2 -b Rocky-10-GenericCloud.qcow2 static-ip-vm.qcow2

    virt-install --name rocky10-static-ip \
    --memory 2048 --vcpus 2 \
    --disk path=static-ip-vm.qcow2,format=qcow2 \
    --cloud-init user-data=user-data.yml,hostname=network-server \
    --os-variant rockylinux10 \
    --import --noautoconsole
    
    # 连接到虚拟控制台
    virsh console rocky10-static-ip

    # 登录后，检查 IP 地址
    [rocky@network-server ~]$ ip a show eth0
    ```

    输出应该显示 `eth0` 具有静态 IP 地址 `192.168.122.100/24`。

### 示例 2：多接口配置

一个典型的真实场景是一台有多个网络接口的服务器。在这里，我们将创建一个有两个接口的虚拟机：`eth0` 将使用 DHCP，`eth1` 将使用静态 IP。

1. **为两个接口创建 `user-data.yml`：**

    ```bash
    cat <<EOF > user-data.yml
    #cloud-config
    packages: [iperf3]

    ---

    network:
      version: 2
      ethernets:
        eth0:
          dhcp4: yes
        eth1:
          dhcp4: no
          addresses: [192.168.200.10/24]
    EOF
    ```

2. **启动一个有两个网卡 (NIC) 的虚拟机：** 我们在 `virt-install` 命令中添加第二个 `--network` 标志。

    ```bash
    virt-install --name rocky10-multi-nic \
    --memory 2048 --vcpus 2 \
    --disk path=... \
    --network network=default,model=virtio \
    --network network=default,model=virtio \
    --cloud-init user-data=user-data.yml,hostname=multi-nic-server \
    --os-variant rockylinux10 --import --noautoconsole
    ```

3. **验证：** 通过 DHCP 分配的地址 SSH 到 `eth0`，然后使用 `ip a show eth1` 检查 `eth1` 上的静态 IP。

## 2. 使用多部分 MIME 统一负载

有时，你需要在主 `#cloud-config` 模块执行 *之前* 运行一个设置脚本。MIME (多用途互联网邮件扩展) 多部分文件是解决方案，它允许你将不同的内容类型捆绑成一个有序的负载。

你可以将 MIME 文件的结构可视化如下：

```
+-----------------------------------------+
| 主头 (multipart/mixed; boundary) |
+-----------------------------------------+
|
| --boundary                              |
| +-------------------------------------+
| | 第一部分头 (例如 text/x-shellscript)  |
| +-------------------------------------+
| | 第一部分内容 (#/bin/sh...)        |
| +-------------------------------------+
|
| --boundary                              |
| +-------------------------------------+
| | 第二部分头 (例如 text/cloud-config)   |
| +-------------------------------------+
| | 第二部分内容 (#cloud-config...)   |
| +-------------------------------------+
|
| --boundary-- (关闭)                  |
+-----------------------------------------+
```

### 动手实践：预检检查脚本

我们将创建一个多部分文件，首先运行一个 Shell 脚本，然后继续执行主 `#cloud-config`。

1. **创建多部分 `user-data.mime` 文件：**

    这是一个特殊格式的文本文件，使用 "boundary" (边界) 字符串来分隔各个部分。

    ```bash
    cat <<EOF > user-data.mime
    Content-Type: multipart/mixed; boundary="//"
    MIME-Version: 1.0

    --//
    Content-Type: text/x-shellscript; charset="us-ascii"

    #!/bin/sh
    echo "Running pre-flight checks..."
    # 在真实脚本中，你可能会检查磁盘空间或内存。
    # 如果检查失败，你可以 'exit 1' 来停止 cloud-init。
    echo "Pre-flight checks passed." > /tmp/pre-flight-status.txt

    --//
    Content-Type: text/cloud-config; charset="us-ascii"

    #cloud-config
    packages:
        - htop
        runcmd:
          - [ sh, -c, "echo 'Main cloud-config ran successfully' > /tmp/main-config-status.txt" ]

    --//--
    EOF
    ```

    !!! note "关于 MIME 边界"

        边界字符串（此处为 `//`）是一个任意字符串，它不能出现在任何部分的内容中。它用于分隔文件的各个部分。

2. **启动并验证：**

    你将此文件传递给 `virt-install` 的方式与标准 `user-data.yml` 文件相同。

    ```bash
    # 使用一个新的磁盘镜像
    qemu-img create -f qcow2 -F qcow2 -b Rocky-10-GenericCloud.qcow2 mime-vm.qcow2

    virt-install --name rocky10-mime-test \
    --memory 2048 --vcpus 2 \
    --disk path=mime-vm.qcow2,format=qcow2 \
    --cloud-init user-data=user-data.mime,hostname=mime-server \
    --os-variant rockylinux10 \
    --import --noautoconsole
    ```

    启动后，通过 SSH 登录虚拟机，通过查找它们创建的文件来检查两个部分是否都已执行：

    ```bash
    cat /tmp/pre-flight-status.txt
    cat /tmp/main-config-status.txt
    ```

!!! tip "其他多部分内容类型"

    `cloud-init` 支持其他用于高级用途的内容类型，例如用于非常早期启动脚本的 `text/cloud-boothook` 或用于运行自定义 Python 代码的 `text/part-handler`。请参考官方文档了解更多细节。

## 下一步

你现在已经学会了两种强大的高级 `cloud-init` 技术。现在你可以定义静态网络，并使用多部分 user-data 编排复杂的配置工作流。

在下一章中，我们将从 *消费* `cloud-init` 的逐个实例视角转向 *自定义其默认行为*，以创建你自己的预配置"黄金镜像" (golden images)。
