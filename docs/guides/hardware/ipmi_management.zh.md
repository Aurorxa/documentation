---
title: IPMI 管理
author: Howard Van Der Wal
contributors: Steven Spencer
ai_contributors: Claude (claude-opus-4-6)
tested with: 8, 9, 10
tags:
- bmc
- hardware
- ipmi
- management
- server
---

**知识水平**：:star: :star: :star:
**阅读时间**：20 分钟

## AI 使用说明

本文档遵循[此处提供的 AI 贡献政策](../contribute/ai-contribution-policy.md)。如果您发现操作说明中有任何错误，请告知我们。

## 简介

IPMI（智能平台管理接口）是一个开放标准，用于独立于操作系统监控和管理服务器硬件^1^。IPMI 与 BMC（基板管理控制器）通信，BMC 是嵌入在服务器主板上的专用微控制器。BMC 拥有自己的固件、网络连接和电源，使管理员能够在主机操作系统无响应或已关机的情况下监控硬件健康状况、控制电源状态并访问远程控制台^2^。

本指南涵盖了在 Rocky Linux 上安装和配置 IPMI 工具、执行常见本地和远程管理操作，以及排查管理员最常遇到的问题。

## 前置条件

- 一台运行 Rocky Linux 8、9 或 10 且配备 BMC/IPMI 兼容硬件的系统（Dell、HPE、Supermicro、Lenovo 等大多数机架式和塔式服务器均包含 BMC）。

- Rocky Linux 主机上的 root 或 `sudo` 访问权限。

- 对 BMC 管理接口的网络访问（用于远程操作）。

- `ipmitool` 或 `freeipmi` 软件包（安装方法见下文）。

## 安装 IPMI 工具

使用 `dnf` 安装 `ipmitool`：

```bash
sudo dnf install ipmitool
```

验证安装：

```bash
ipmitool -V
```

## 加载 IPMI 内核模块

Rocky Linux 默认不加载 IPMI 内核模块。在本地使用 `ipmitool` 之前，必须先加载所需的模块。

!!! warning

    如果未加载 IPMI 内核模块，`ipmitool` 将失败并报错：`Could not open device at /dev/ipmi0 or /dev/ipmi/0 or /dev/ipmidev/0: No such file or directory`。这是 Rocky Linux 上最常见的 IPMI 问题。

加载三个核心 IPMI 模块：

```bash
sudo modprobe ipmi_msghandler
sudo modprobe ipmi_devintf
sudo modprobe ipmi_si
```

- `ipmi_msghandler` — 核心 IPMI 消息处理器，为 IPMI 通信提供框架^3^。
- `ipmi_devintf` — 创建 `/dev/ipmi0` 字符设备，用户空间工具（如 `ipmitool`）通过该设备与 BMC 通信^3^。
- `ipmi_si` — 系统接口驱动，通过 KCS（键盘控制器风格）、SMIC 或 BT 接口与 BMC 通信^3^。

验证模块已加载：

```bash
lsmod | grep ipmi
```

配备 BMC 硬件的服务器上的预期输出：

```text
ipmi_si                73728  0
ipmi_devintf           20480  0
ipmi_msghandler       106496  2 ipmi_devintf,ipmi_si
```

## 使 IPMI 模块在重启后持久加载

使用 `modprobe` 加载的模块在重启后会失效。要在启动时自动加载，请在 `/etc/modules-load.d/` 中创建配置文件：

```bash
cat << "EOF" | sudo tee /etc/modules-load.d/ipmi.conf
ipmi_msghandler
ipmi_devintf
ipmi_si
EOF
```

创建此文件后，`systemd-modules-load` 服务将在每次启动时加载这些模块。

## 验证 IPMI 设备

加载模块后，验证 `/dev/ipmi0` 设备是否存在：

```bash
ls -la /dev/ipmi*
```

检查内核环形缓冲区中的 IPMI 相关消息：

```bash
dmesg | grep -i ipmi
```

在配备 BMC 硬件的服务器上，您将看到指示 IPMI 系统接口已被找到且设备已注册的消息。

测试与 BMC 的连通性：

```bash
sudo ipmitool mc info
```

这将显示 BMC 固件版本、制造商和其他管理控制器详细信息。

## 本地和远程 IPMI 访问

### 本地访问

本地访问使用 `/dev/ipmi0` 设备直接与同一台机器上的 BMC 通信：

```bash
sudo ipmitool <command>
```

### 远程访问

远程访问通过网络使用 IPMI LAN 接口连接到 BMC。这对于从中央管理站或头节点管理服务器非常有用：

```bash
ipmitool -H <BMC_IP> -I lanplus -U <username> -P <password> <command>
```

- `-H` — BMC IP 地址或主机名。
- `-I lanplus` — 使用 IPMI v2.0 RMCP+（LAN Plus）接口，该接口提供身份验证和加密^1^。
- `-U` — BMC 用户名。
- `-P` — BMC 密码。

!!! warning

    在命令行中传递密码会在进程列表和 shell 历史记录中暴露密码。对于脚本化或生产环境使用，请考虑使用 `-f` 标志从文件读取密码，或使用 `-E` 标志从 `IPMI_PASSWORD` 环境变量读取密码。

检查远程服务器电源状态的示例：

```bash
ipmitool -H 192.168.1.100 -I lanplus -U admin -P password chassis power status
```

## 常见 ipmitool 操作

### 传感器读数

查看所有传感器数据及其阈值：

```bash
sudo ipmitool sensor list
```

查看传感器数据仓库（SDR）的简要摘要：

```bash
sudo ipmitool sdr
```

### 电源控制

```bash
sudo ipmitool chassis power status
sudo ipmitool chassis power on
sudo ipmitool chassis power off
sudo ipmitool chassis power cycle
sudo ipmitool chassis power reset
```

- `power cycle` — 先关闭服务器再重新打开（硬电源循环）。
- `power reset` — 执行硬件重置，不进行完整的电源循环。

### 机箱状态

```bash
sudo ipmitool chassis status
```

这将显示当前电源状态、上一次电源事件以及各种机箱状态标志。

### SEL（系统事件日志）

SEL 记录硬件事件，如温度阈值超标、风扇故障和内存错误：

```bash
sudo ipmitool sel list
```

要获得更详细、更易读的输出：

```bash
sudo ipmitool sel elist
```

在查看事件后清除 SEL：

```bash
sudo ipmitool sel clear
```

!!! warning

    仅当您查看并记录了所有重要事件后才清除 SEL。一旦清除，事件将无法恢复。

### SOL（局域网串口）

SOL 将服务器的串行控制台重定向到 IPMI LAN 接口，即使在网络协议栈不可用时也能提供远程控制台访问：

```bash
ipmitool -H <BMC_IP> -I lanplus -U <username> -P <password> sol activate
```

断开 SOL 会话：

```bash
ipmitool -H <BMC_IP> -I lanplus -U <username> -P <password> sol deactivate
```

也可以在客户端按 `~.` 来终止 SOL 会话。

### 用户管理

列出 BMC 通道 1 上配置的所有用户：

```bash
sudo ipmitool user list 1
```

## 排查 /dev/ipmi0 缺失问题

本节解决最常见的问题：`ipmitool` 因 `/dev/ipmi0` 不存在而失败。

### 步骤 1 — 检查 IPMI 模块是否已加载

```bash
lsmod | grep ipmi
```

如果没有输出，请按上文"加载 IPMI 内核模块"部分所述加载模块。

### 步骤 2 — 检查 dmesg 中是否有错误

```bash
dmesg | grep -i ipmi
```

查找以下消息：

- `ipmi_si: Unable to find any System Interface(s)` — 表示内核无法检测到 BMC。请验证硬件是否配备 BMC 以及 BIOS/UEFI 设置中是否启用了 IPMI。
- `ipmi_si: Trying KCS-defined... success` — 接口已成功找到。

### 步骤 3 — 重启 IPMI 服务

如果已安装 OpenIPMI 服务，请重启它：

```bash
sudo systemctl restart ipmi
```

如果该服务不存在，可以安装它：

```bash
sudo dnf install OpenIPMI
sudo systemctl enable --now ipmi
```

### 步骤 4 — 验证 BIOS/UEFI 设置

如果模块已加载但未检测到 BMC：

- 启动时进入 BIOS/UEFI 设置。
- 导航到 IPMI 或 BMC 配置部分。
- 确保已启用 KCS 上的 IPMI。
- 保存更改并重启。

!!! warning

    虚拟机和云实例通常没有物理 BMC 硬件。IPMI 模块可以加载，但 `ipmitool` 将报告未找到 BMC。IPMI 需要配备嵌入式 BMC 的物理服务器硬件。

### 步骤 5 — 检查驱动冲突

在某些系统上，`ipmi_si` 模块可能与厂商特定的管理驱动冲突。如果在 `dmesg` 中看到错误，请尝试卸载并重新加载模块：

```bash
sudo modprobe -r ipmi_si ipmi_devintf ipmi_msghandler
sudo modprobe ipmi_msghandler
sudo modprobe ipmi_devintf
sudo modprobe ipmi_si
```

## FreeIPMI 作为替代方案

FreeIPMI 是另一种 IPMI 实现，提供自己的命令行工具集^4^。一些管理员更偏好 FreeIPMI，因为它具有详细的输出格式和额外的功能。

### 安装

```bash
sudo dnf install freeipmi
```

### 主要 FreeIPMI 命令

| FreeIPMI 命令 | ipmitool 等效命令 | 描述 |
| --- | --- | --- |
| `bmc-info` | `ipmitool mc info` | 显示 BMC 信息 |
| `ipmi-sensors` | `ipmitool sensor list` | 列出所有传感器读数 |
| `ipmi-sel` | `ipmitool sel list` | 显示 SEL（系统事件日志） |
| `ipmi-chassis --get-chassis-status` | `ipmitool chassis status` | 显示机箱状态 |
| `ipmipower --stat` | `ipmitool chassis power status` | 检查电源状态 |
| `ipmipower --on` | `ipmitool chassis power on` | 开启服务器电源 |
| `ipmipower --off` | `ipmitool chassis power off` | 关闭服务器电源 |

FreeIPMI 工具也支持通过 `-h`、`-u` 和 `-p` 标志进行远程访问：

```bash
bmc-info -h <BMC_IP> -u <username> -p <password>
```

## BMC 固件更新注意事项

!!! danger

    BMC 固件更新存在风险。更新失败或中断可能导致 BMC 无响应，可能需要更换物理主板。请务必严格遵循硬件厂商的说明。

BMC 固件更新的一般指导原则：

- 仅从服务器厂商的官方支持网站（Dell、HPE、Supermicro、Lenovo）下载固件。
- 在应用更新前阅读固件版本的发布说明。
- 确保在整个更新过程中供电稳定。如有 UPS（不间断电源），请使用。
- 在固件更新过程中不要重启或关闭服务器。
- 如果厂商工具支持，请在更新前备份当前的 BMC 配置。

大多数厂商提供自己的 BMC 固件更新工具：

- Dell：`racadm` 或 Dell EMC Repository Manager。
- HPE：`ilorest` 或 HPE Smart Update Manager。
- Supermicro：`sum`（Supermicro Update Manager）或基于 Web 的 BMC 界面。
- Lenovo：`OneCLI` 或 Lenovo XClarity。

## 总结

您现在已经在 Rocky Linux 上拥有一个可用的 IPMI 配置，用于本地和远程管理服务器硬件。涵盖的关键步骤包括加载和持久化所需的内核模块^3^、使用 `ipmitool` 进行常见管理任务^1^、排查最常见的 `/dev/ipmi0` 问题，以及使用 FreeIPMI 作为替代方案^4^。

要深入了解 IPMI 规范和高级功能，请参阅 Intel IPMI 规范^1^和 `ipmitool` 仓库^2^。

## 参考资料

1. "Intelligent Platform Management Interface Specification, Second Generation, v2.0"，Intel Corporation [https://www.intel.com/content/dam/www/public/us/en/documents/product-briefs/ipmi-second-gen-interface-spec-v2-rev1-1.pdf](https://www.intel.com/content/dam/www/public/us/en/documents/product-briefs/ipmi-second-gen-interface-spec-v2-rev1-1.pdf)
2. "ipmitool — utility for controlling IPMI-enabled devices"，Duncan Laurie 及贡献者 [https://github.com/ipmitool/ipmitool](https://github.com/ipmitool/ipmitool)
3. "IPMI — The Linux Kernel documentation"，Corey Minyard [https://www.kernel.org/doc/html/latest/driver-api/ipmi.html](https://www.kernel.org/doc/html/latest/driver-api/ipmi.html)
4. "FreeIPMI — GNU Project"，Albert Chu 及 GNU FreeIPMI 团队 [https://www.gnu.org/software/freeipmi/](https://www.gnu.org/software/freeipmi/)
