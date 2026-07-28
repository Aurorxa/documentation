---
title: IRQ 与内核数据包丢失
author: Howard Van Der Wal
ai_contributors: Claude (claude-opus-4-6)
tested with: 8.10, 9.7, 10.1
tags:
- network
- performance
- irq
- tuning
---

## AI 使用说明

本文档遵循[此处提供的 AI 贡献政策](../../contribute/ai-contribution-policy.md)。如果您发现操作说明中有任何错误，请告知我们。

## 简介

在承受重网络负载的服务器上，即使硬件有能力处理流量，也可能出现数据包丢失、延迟峰值和吞吐量下降。这些问题通常源于 Linux 内核如何在 CPU 之间分配网络中断处理，而非 NIC（网络接口卡）本身的问题。

本指南涵盖了在 Rocky Linux 上诊断和解决常见网络性能问题，包括：

- 区分内核级数据包丢失与 NIC 硬件丢包。
- 识别并纠正跨 CPU 的 IRQ（中断请求）不平衡。
- 调优 NIC 环形缓冲区、通道数量和传输队列长度。
- 配置 irqbalance 以实现最佳中断分配。
- 使所有更改在重启后持久化。

这些技术适用于裸金属服务器以及带有半虚拟化或 SR-IOV 网络接口的虚拟机。诊断命令适用于任何 Rocky Linux 系统，而某些调优操作需要硬件支持。

## 前置条件

在开始之前，请确保您具有：

- 一台运行 Rocky Linux 8.10、9.x 或 10.x 的系统
- Root 或 sudo 访问权限。
- 已安装 `ethtool` 软件包（`dnf install ethtool`）。
- 已安装 `irqbalance` 软件包（`dnf install irqbalance`）。
- 已安装 `sysstat` 软件包以使用 `mpstat`（`dnf install sysstat`）。
- 基本熟悉 Linux 网络和命令行。

一次性安装所有必需的软件包：

```bash
dnf install ethtool irqbalance sysstat iproute
```

## 理解数据包丢失

Linux 上的数据包丢失可能发生在两个层级：NIC 硬件和内核软件栈。确定丢失发生的位置是解决问题的第一步。

### 检查丢包计数器

使用 `ip -s link show` 查看接口统计信息：

```bash
ip -s link show eth0
```

输出包括 RX 和 TX 统计信息。查看 RX 行中的 `dropped` 计数器：

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 56:00:04:b2:a5:f1 brd ff:ff:ff:ff:ff:ff
    RX:  bytes packets errors dropped  missed  mcast
    1234567890 9876543   0      1523      0       0
    TX:  bytes packets errors dropped carrier collsns
    9876543210 8765432   0       0       0       0
```

对于 NIC 级统计信息，使用 `ethtool -S`^5^ 并过滤丢包计数器：

```bash
ethtool -S eth0 | grep -i drop
```

示例输出：

```text
     rx_dropped: 1523
     tx_dropped: 0
     rx_dropped.nic: 0
     tx_dropped_link_down.nic: 0
```

!!! note "计数器名称因驱动而异"

    确切的计数器名称取决于 NIC 驱动。Intel `ice` 和 `i40e` 驱动报告 `rx_dropped` 和 `rx_dropped.nic`。Virtio（KVM/云）驱动报告每个队列的计数器，如 `rx_queue_0_drops`。Broadcom `bnxt_en` 驱动使用 `rx_discard_pkts`。请查阅您的驱动文档以获取具体的计数器名称。

### 解释丢包计数器

`rx_dropped` 与 `rx_dropped.nic` 之间的区别至关重要：

- `rx_dropped.nic: 0` 表示 NIC 硬件成功接收了所有数据包。NIC 环形缓冲区未溢出。
- `rx_dropped: 1523` 表示内核在 NIC 传递后丢弃了 1523 个数据包。内核软件栈未能足够快地处理它们。

当 `rx_dropped.nic` 为零但 `rx_dropped` 在增长时，问题出在内核而非硬件。这种模式指向 IRQ 不平衡、环形缓冲区不足或 CPU 饱和导致无法及时处理数据包。

!!! warning "不要将内核丢包与 NIC 丢包混淆"

    非零的 `rx_dropped` 加上 `rx_dropped.nic: 0` 并不表示硬件问题。更换线缆、SFP 收发器或 NIC 无法解决内核级丢包。应专注于 IRQ 分发和内核调优。

## 诊断 IRQ 不平衡

网络接口卡生成硬件中断 (IRQ) 来通知内核有数据包到达。每个 NIC 队列都有自己的 IRQ^1^，内核将每个 IRQ 分配到特定的 CPU。当太多 IRQ 落在同一个 CPU 上时，该 CPU 成为瓶颈，而其他 CPU 处于空闲状态。

### 读取 /proc/interrupts

查看当前中断分布：

```bash
cat /proc/interrupts | head -1; cat /proc/interrupts | grep eth0
```

每行显示一个 IRQ 编号和每个 CPU 处理的中断计数。寻找一个 CPU 处理的中断数量明显多于其他的巨大差异。

要总结特定接口的分布，统计每个 CPU 分配了多少 IRQ：

```bash
grep eth0 /proc/interrupts | awk '{irq=$1; sub(/:$/,"",irq); max=0; maxcpu="N/A"; for(i=2; i<=NF; i++) {if($i ~ /^[0-9]+$/ && $i+0 > max) {max=$i+0; maxcpu=i-2}} print "IRQ " irq " -> CPU " maxcpu " (" max " interrupts)"}'
```

### 检查 CPU 利用率

使用 `mpstat` 识别被中断处理饱和的 CPU：

```bash
mpstat -P ALL 1 5
```

寻找 `%irq` 或 `%soft`（softIRQ）利用率高而其他 CPU 显示接近零值的 CPU。这表示 IRQ 不平衡。

### 检查 softnet 统计信息

文件 `/proc/net/softnet_stat` 提供每个 CPU 的网络处理统计信息。每行对应一个 CPU（从 CPU 0 开始）：

```bash
cat /proc/net/softnet_stat
```

输出为十六进制，包含以下列：

| 列 | 字段        | 含义                                            |
| ------ | ------------ | -------------------------------------------------- |
| 1      | processed    | 该 CPU 处理的数据包总数                |
| 2      | dropped      | 丢弃的数据包（积压溢出）                 |
| 3      | time_squeeze | CPU 在完成前耗尽预算的次数   |

非零的 `time_squeeze` 值（第 3 列）表示内核在处理完所有待处理数据包之前耗尽了 `netdev_budget`。这会导致数据包在环形缓冲区中滞留更长时间，在突发负载下可能导致丢包。

以可读格式查看统计信息：

```bash
awk '{printf "CPU%-4d processed=%-12d dropped=%-8d time_squeeze=%d\n", NR-1, strtonum("0x"$1), strtonum("0x"$2), strtonum("0x"$3)}' /proc/net/softnet_stat
```

### 检查当前 IRQ 亲和性

内核通过 `/proc/irq/` 接口公开 IRQ 亲和性设置^2^。要查看 IRQ 当前分配到哪个 CPU：

```bash
cat /proc/irq/45/smp_affinity_list
```

将 `45` 替换为 `/proc/interrupts` 中的实际 IRQ 编号。输出显示哪个或哪些 CPU 处理该 IRQ。

列出与某个接口关联的所有 IRQ 的亲和性：

```bash
for irq in $(grep eth0 /proc/interrupts | awk '{print $1}' | tr -d ':'); do
    echo "IRQ $irq -> CPU $(cat /proc/irq/$irq/smp_affinity_list)"
done
```

## 理解 APIC 向量耗尽

每个 CPU 有数量有限的 APIC 向量（每个 CPU 约 200 个可用）用于处理硬件中断。每个支持 MSI/MSI-X^3^ 的 PCI 设备在其目标 CPU 上消耗一个或多个向量。

### 向量如何被耗尽

配备多个高速 NIC、存储控制器和 SR-IOV 虚拟功能的现代服务器可能消耗数百个向量。当 CPU 没有剩余空闲向量时，内核无法将新的 IRQ 分配到该 CPU。这会强制 IRQ 落到任何仍有可用向量的 CPU 上，造成严重不平衡。

### 向量耗尽的症状

- IRQ 亲和性脚本（如 `set_irq_affinity`）失败或产生意外结果。
- 尽管 irqbalance 在运行，网络 IRQ 仍聚集在少数几个 CPU 上。
- `/proc/interrupts` 显示极端偏斜，大多数 CPU 处理零个网络中断。

!!! note "向量耗尽不限于 SR-IOV"

    任何支持 MSI/MSI-X 的 PCI 设备都会消耗 APIC 向量，包括存储控制器、GPU 加速器和 InfiniBand 适配器。即使没有启用 SR-IOV，带有许多 PCI 设备的服务器也可能经历向量耗尽。

!!! note "Rocky Linux 9 内核改进"

    Rocky Linux 9 搭载内核 5.14 及更高版本，包含 Linux 4.15 中引入的 APIC 向量管理改进。

### 减少向量消耗

减少向量消耗的最有效方法是降低 NIC 队列数量。每个组合队列消耗一个 IRQ 和一个 APIC 向量。将 NIC 从 128 个组合队列减少到 64 个，可将其向量消耗减半。见下文"调优 NIC 通道数量"部分。

## 配置 irqbalance

`irqbalance`^4^ 守护进程自动在 CPU 之间分配硬件中断。它在 Rocky Linux 上默认运行，通常是中断管理的正确起点。

### 验证 irqbalance 状态

```bash
systemctl status irqbalance
```

如果 irqbalance 未运行，请启用并启动它：

```bash
systemctl enable --now irqbalance
```

### 配置文件

irqbalance 配置文件位于 `/etc/sysconfig/irqbalance`。关键设置包括：

- **IRQBALANCE_BANNED_CPULIST**：逗号分隔的 CPU 列表或范围，irqbalance 不应将 IRQ 分配到这些 CPU。这对于隔离专用于实时或延迟敏感工作负载的 CPU 很有用。

例如，防止 irqbalance 将 IRQ 分配到 CPU 8-63：

```bash
# /etc/sysconfig/irqbalance
IRQBALANCE_BANNED_CPULIST=8-63
```

更改配置后，重启 irqbalance：

```bash
systemctl restart irqbalance
```

### 验证中断分布

重启 irqbalance 后，等待 10-30 秒让其重新分配 IRQ，然后检查分布：

```bash
for irq in $(grep eth0 /proc/interrupts | awk '{print $1}' | tr -d ':'); do
    echo "IRQ $irq -> CPU $(cat /proc/irq/$irq/smp_affinity_list)"
done
```

!!! warning "过度限制 CPU 禁止可能加剧不平衡"

    禁止过多 CPU 会强制所有 IRQ 落到少数剩余的 CPU 上。如果未被禁止的 CPU 也用完了 APIC 向量，irqbalance 无法有效分配 IRQ。仅禁止真正需要隔离的 CPU，并在更改后验证结果分布。

## 手动重新分配 IRQ 亲和性

当 irqbalance 无法达到足够的分布（例如，由于 APIC 向量约束），可以手动将 IRQ 绑定到特定的 CPU。

### 通过 /proc 设置 IRQ 亲和性

将特定 IRQ 分配到特定 CPU：

```bash
echo 4 > /proc/irq/45/smp_affinity_list
```

这将 IRQ 45 分配到 CPU 4。将数字替换为实际 IRQ 和目标 CPU。

### 将 IRQ 分布到多个 CPU

以下脚本将接口的所有 IRQ 均匀分布到一组 CPU 上：

```bash
#!/bin/bash
IFACE="eth0"
CPUS=(0 1 2 3 4 5 6 7)
IDX=0

for IRQ in $(grep "$IFACE" /proc/interrupts | awk '{print $1}' | tr -d ':'); do
    CPU=${CPUS[$IDX]}
    echo "$CPU" > /proc/irq/$IRQ/smp_affinity_list
    echo "IRQ $IRQ -> CPU $CPU"
    IDX=$(( (IDX + 1) % ${#CPUS[@]} ))
done
```

保存此脚本并以 root 权限运行。调整 `CPUS` 数组以匹配要处理网络中断的 CPU。

### 感知 NUMA 的 IRQ 放置

为获得最佳性能，将 NIC IRQ 分配到与 NIC 位于同一 NUMA 节点上的 CPU。确定 NIC 的 NUMA 节点：

```bash
cat /sys/class/net/eth0/device/numa_node
```

然后列出该 NUMA 节点上的 CPU：

```bash
lscpu | grep "NUMA node"
```

仅将 IRQ 分配到匹配 NUMA 节点上的 CPU，以避免数据包处理期间的跨节点内存访问。

!!! note "NUMA 与虚拟机"

    虚拟机和云实例可能不会公开 NUMA 拓扑或 `/sys/class/net/<iface>/device/numa_node` 路径。感知 NUMA 的 IRQ 放置最适用于带有多个 CPU 插槽的裸金属服务器。

## 调优 NIC 环形缓冲区

环形缓冲区是 NIC 中的固定大小队列，传入的数据包在其中等待内核处理。较大的环形缓冲区为内核处理数据包提供了更多时间，防止缓冲区溢出和丢包。

### 检查当前环形缓冲区大小

```bash
ethtool -g eth0
```

示例输出：

```text
Ring parameters for eth0:
Pre-set maximums:
RX:         4096
TX:         4096
Current hardware settings:
RX:         256
TX:         256
```

### 增大环形缓冲区

将环形缓冲区设置为支持的最大值：

```bash
ethtool -G eth0 rx 4096 tx 4096
```

将 `4096` 替换为 `ethtool -g` 输出中"Pre-set maximums"部分显示的 NIC 最大值。

!!! warning "环形缓冲区更改与内存"

    较大的环形缓冲区每个队列消耗更多的内核内存。在具有 64 个组合队列且每个队列 4096 条目的 NIC 上，这可能使用数百兆字节的内存。在内存有限的系统上，请逐步增加环形缓冲区并监控内存使用情况。

### 持久化环形缓冲区更改

环形缓冲区设置在重启后会重置。要持久化它们，创建 NetworkManager 调度脚本：

```bash
cat > /etc/NetworkManager/dispatcher.d/20-ring-buffers <<'SCRIPT'
#!/bin/bash
IFACE=$1
ACTION=$2

if [ "$ACTION" = "up" ] && [ "$IFACE" = "eth0" ]; then
    ethtool -G eth0 rx 4096 tx 4096
fi
SCRIPT
chmod +x /etc/NetworkManager/dispatcher.d/20-ring-buffers
```

## 调优传输队列长度

传输队列长度 (`txqueuelen`) 控制内核在丢弃新数据包之前可以排队等待传输的数据包数量。默认值通常为 1000。

### 何时增大 txqueuelen

在操作系统升级或内核更改后，默认传输行为可能发生变化。如果观察到 TX 丢包或应用报告的数据包丢失在增加队列长度后解决，调优 `txqueuelen` 可能有所帮助。

### 设置 txqueuelen

检查当前值：

```bash
ip link show eth0 | grep qlen
```

设置新值：

```bash
ip link set eth0 txqueuelen 5000
```

安全值范围从 1000 到 20000。从默认值 (1000) 开始，仅在监控确认有 TX 丢包时才增加。

!!! warning "避免设置过高的 txqueuelen"

    高于 20000 的值可能导致缓冲区膨胀 (bufferbloat)，即数据包在队列中停留时间过长，到达目的地时已太迟而无用。这表现为负载下的高延迟，尽管吞吐量看似正常。如果提高 txqueuelen 后延迟增加，请降低该值。

### 使用 NetworkManager 调度脚本持久化 txqueuelen

创建在接口启用时运行的调度脚本：

```bash
cat > /etc/NetworkManager/dispatcher.d/30-txqueuelen <<'SCRIPT'
#!/bin/bash
IFACE=$1
ACTION=$2

if [ "$ACTION" = "up" ] && [ "$IFACE" = "eth0" ]; then
    ip link set eth0 txqueuelen 5000
fi
SCRIPT
chmod +x /etc/NetworkManager/dispatcher.d/30-txqueuelen
```

通过重启接口验证脚本是否正常工作：

```bash
nmcli connection down eth0 && nmcli connection up eth0
ip link show eth0 | grep qlen
```

### 使用 udev 规则持久化 txqueuelen

作为 NetworkManager 调度脚本的替代方案，使用 udev 规则：

```bash
echo 'ACTION=="add", SUBSYSTEM=="net", KERNEL=="eth0", ATTR{tx_queue_len}="5000"' > /etc/udev/rules.d/90-txqueuelen.rules
```

重新加载 udev 规则：

```bash
udevadm control --reload-rules
```

## 调优 NIC 通道数量

NIC 通道（也称为队列）确定 NIC 提供多少条独立的数据包处理路径。每个通道获得自己的 IRQ。更多通道允许更多并行处理，但也消耗更多 APIC 向量。

### 检查当前通道

```bash
ethtool -l eth0
```

示例输出：

```text
Channel parameters for eth0:
Pre-set maximums:
RX:         0
TX:         0
Other:      0
Combined:   4
Current hardware settings:
RX:         0
TX:         0
Other:      0
Combined:   2
```

### 调整通道数量

减少组合队列（例如，从 128 个减少到 64 个以降低 IRQ 数量）：

```bash
ethtool -L eth0 combined 2
```

!!! note "将通道与 CPU 匹配"

    常见做法是将组合通道数量设置为与可用于网络处理的 CPU 数量相等。拥有的队列多于可用 CPU 意味着某些队列将共享 CPU，这不会带来性能益处，同时会消耗额外的 APIC 向量。

### 为什么更少的队列可以改善性能

在面临 APIC 向量压力的系统上，减少队列数量会降低总中断数量。这使内核能够将剩余的 IRQ 更均匀地分布在 CPU 之间。例如，在两个接口上将组合队列从 128 个减少到 64 个，可将总 IRQ 数量从 256 个降低到 128 个，显著缓解向量压力。

## 监控网络性能

定期监控有助于在问题影响应用之前识别问题。

### 接口统计信息

实时监控接口统计信息：

```bash
watch -n 1 'ip -s link show eth0'
```

### NIC 级计数器

查看所有 NIC 统计信息：

```bash
ethtool -S eth0
```

过滤特定计数器：

```bash
ethtool -S eth0 | grep -E 'rx_dropped|tx_dropped|rx_error|tx_error'
```

### softnet 统计信息

监控 softnet 统计信息的随时间变化：

```bash
watch -n 1 'awk "{printf \"CPU%-4d processed=%-12d dropped=%-8d time_squeeze=%d\n\", NR-1, strtonum(\"0x\"\$1), strtonum(\"0x\"\$2), strtonum(\"0x\"\$3)}" /proc/net/softnet_stat'
```

### 有用的 sysctl 可调参数

两个 sysctl 参数控制内核处理网络数据包的激进程度：

- **net.core.netdev_budget**：内核在一个 softIRQ 周期中处理的最大数据包数（默认：300）
- **net.core.netdev_budget_usecs**：一个 softIRQ 周期的最大微秒数（默认：2000）

检查当前值：

```bash
sysctl net.core.netdev_budget
sysctl net.core.netdev_budget_usecs
```

如果 `/proc/net/softnet_stat` 显示频繁的 time squeeze，请增加预算：

```bash
sysctl -w net.core.netdev_budget=600
sysctl -w net.core.netdev_budget_usecs=4000
```

另一个有用的可调参数是 `netdev_max_backlog`，它控制每个 CPU 的积压队列大小：

```bash
sysctl net.core.netdev_max_backlog
```

如果 `/proc/net/softnet_stat` 中的 dropped 列为非零，请增加积压：

```bash
sysctl -w net.core.netdev_max_backlog=5000
```

## LACP 绑定接口注意事项

在调优绑定^6^接口的网络性能时，有几个额外因素需要考虑。

### IRQ 调优针对从属接口

IRQ 亲和性、环形缓冲区和通道数量更改适用于各个从属接口（如 `eth0` 和 `eth1`），而非绑定接口 (`bond0`)。绑定接口是软件构造，没有自己的 IRQ 或环形缓冲区。

```bash
# 正确：调优从属接口
ethtool -G eth0 rx 4096 tx 4096
ethtool -G eth1 rx 4096 tx 4096

# 这对环形缓冲区无效：
# ethtool -G bond0 rx 4096 tx 4096
```

### 传输哈希策略

传输哈希策略决定出站流量如何在绑定从属接口之间分配。检查当前策略：

```bash
cat /proc/net/bonding/bond0 | grep "Transmit Hash"
```

常见策略：

- `layer2` — 基于 MAC 地址哈希（默认）。
- `layer2+3` — 基于 MAC 和 IP 地址哈希（对路由流量分布更好）。
- `layer3+4` — 基于 IP 地址和端口哈希（分布最佳但可能重新排序数据包）。

### 监控绑定状态

查看绑定状态和从属接口健康状况：

```bash
cat /proc/net/bonding/bond0
```

检查每个从属接口的 `Link Failure Count`。非零值表示历史上发生过链路抖动事件，需要调查物理线缆、SFP 收发器或交换机端口。

### 链路故障检测

在 LACP 模式下，`miimon` 参数控制绑定驱动检查链路状态的频率。默认值 (100ms) 适用于大多数环境。如果链路故障未被及时检测到，请降低该值：

```bash
# 检查当前 miimon 设置
cat /proc/net/bonding/bond0 | grep "MII Polling"
```

## 使更改在重启后持久化

使用 `sysctl`、`ethtool` 和 `ip link` 命令进行的网络调优更改在重启后会丢失。使用以下方法使它们永久生效。

### 用于内核参数的 sysctl.d

为网络调优参数创建配置文件：

```bash
cat > /etc/sysctl.d/99-network-tuning.conf <<'EOF'
# Increase softIRQ processing budget
net.core.netdev_budget = 600
net.core.netdev_budget_usecs = 4000

# Increase per-CPU backlog queue
net.core.netdev_max_backlog = 5000
EOF
```

无需重启即可应用：

```bash
sysctl --system
```

### NetworkManager 调度脚本

`/etc/NetworkManager/dispatcher.d/` 中的调度脚本^7^在接口状态变化时自动运行。它们是 Rocky Linux 上持久化接口特定设置的推荐方法。

上述环形缓冲区和 txqueuelen 部分中的脚本演示了此方法。确保脚本具有可执行权限，并使用数字前缀命名以控制顺序：

```bash
ls -la /etc/NetworkManager/dispatcher.d/
```

### udev 规则

对于无论 NetworkManager 如何都应应用的设备级设置，使用 `/etc/udev/rules.d/` 中的 udev 规则：

```bash
# 示例：为特定接口设置 txqueuelen
echo 'ACTION=="add", SUBSYSTEM=="net", KERNEL=="eth0", ATTR{tx_queue_len}="5000"' > /etc/udev/rules.d/90-net-tuning.rules
udevadm control --reload-rules
```

### tuned 配置文件

`tuned`^8^ 守护进程提供应用一系列调优参数的配置文件。Rocky Linux 包含多个面向网络的配置文件：

```bash
tuned-adm list | grep -i network
```

`network-throughput` 和 `network-latency` 配置文件应用标准网络优化。应用配置文件：

```bash
tuned-adm profile network-throughput
```

验证配置文件是否处于活跃状态且所有设置是否匹配：

```bash
tuned-adm active
tuned-adm verify
```

!!! warning "Tuned 配置文件漂移"

    如果 `tuned-adm verify` 报告当前设置与配置文件不同，则手动更改或其他服务已覆盖了 tuned 设置。重新应用配置文件并调查哪个服务在修改设置。

## 总结

Rocky Linux 上的网络性能问题通常来自内核如何分配和处理网络中断的软件级瓶颈，而非硬件故障。诊断和调优工作流程按以下顺序进行：

1. 确定丢包发生的位置——使用 `ethtool -S` 确定丢包是在内核级别（`rx_dropped` 非零且 `rx_dropped.nic: 0`）还是 NIC 级别
2. 检查 IRQ 分布——读取 `/proc/interrupts` 和 `/proc/net/softnet_stat` 以发现不平衡和 time squeeze
3. 减少中断数量——使用 `ethtool -L` 降低组合队列数量以缓解 APIC 向量压力
4. 增加缓冲区容量——使用 `ethtool -G` 增大环形缓冲区，使用 `ip link set` 增加传输队列长度
5. 重新分配 IRQ——配置 irqbalance 或手动绑定 IRQ 以将负载均匀分布到 CPU 上
6. 调优内核参数——通过 sysctl 调整 `netdev_budget` 和 `netdev_max_backlog`
7. 持久化所有更改——使用 sysctl.d 文件、NetworkManager 调度脚本和 tuned 配置文件以在重启后保留
8. 持续监控——监控丢包计数器和 softnet 统计信息的随时间变化以验证改进效果

## 参考资料

1. "Scaling in the Linux Networking Stack"，the Linux Kernel Community [https://www.kernel.org/doc/html/latest/networking/scaling.html](https://www.kernel.org/doc/html/latest/networking/scaling.html)
2. "SMP IRQ Affinity"，the Linux Kernel Community [https://www.kernel.org/doc/html/latest/core-api/irq/irq-affinity.html](https://www.kernel.org/doc/html/latest/core-api/irq/irq-affinity.html)
3. "MSI/MSI-X Support in Linux"，the Linux Kernel Community [https://www.kernel.org/doc/html/latest/PCI/msi-howto.html](https://www.kernel.org/doc/html/latest/PCI/msi-howto.html)
4. "irqbalance"，the irqbalance Project [https://github.com/Irqbalance/irqbalance](https://github.com/Irqbalance/irqbalance)
5. "ethtool(8) - Linux Man Page"，the Linux Kernel Community [https://man7.org/linux/man-pages/man8/ethtool.8.html](https://man7.org/linux/man-pages/man8/ethtool.8.html)
6. "Linux Ethernet Bonding Driver"，the Linux Kernel Community [https://www.kernel.org/doc/html/latest/networking/bonding.html](https://www.kernel.org/doc/html/latest/networking/bonding.html)
7. "NetworkManager Dispatcher Scripts"，the NetworkManager Project [https://networkmanager.dev/docs/api/latest/NetworkManager-dispatcher.html](https://networkmanager.dev/docs/api/latest/NetworkManager-dispatcher.html)
8. "TuneD"，the TuneD Project [https://tuned-project.org/](https://tuned-project.org/)
