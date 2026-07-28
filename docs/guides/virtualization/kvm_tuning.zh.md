---
title: KVM 调优
author: Howard Van Der Wal
tested_with: 8, 9, 10
ai_contributors: Claude (claude-opus-4-6)
tags:
  - kvm
  - libvirt
  - numa
  - performance
  - virtualization
---

## AI 使用说明

本文档遵循[此处提供的 AI 贡献政策。](../contribute/ai-contribution-policy.md) 如果你在说明中发现任何错误，请告诉我们。

## 简介

本指南涵盖 Rocky Linux 上 KVM/libvirt 虚拟机的高级性能调优。它针对常见的生产环境问题，包括 NUMA (非一致性内存访问) 内存错误分配和 vCPU 调度争用。

涵盖的主题包括：

- NUMA 拓扑感知和内存放置。
- NUMA 系统上 `vm.min_free_kbytes` 的危险。
- 使用 libvirt domain XML 进行 vCPU 绑定 (vCPU pinning)。
- 使用 `isolcpus` 对专用虚拟机工作负载进行 CPU 隔离。
- 虚拟化主机的 `tuned` 配置文件。
- 使用 `virsh` 的 NUMA 感知虚拟机放置。

## 前提条件

- 一台运行 Rocky Linux 8、9 或 10 并已安装配置 KVM/libvirt 的机器。请参阅[在 Rocky Linux 上设置 libvirt](libvirt-rocky.md) 指南进行初始设置。
- 在 hypervisor 宿主机上拥有 root 或 `sudo` 访问权限。
- 熟悉 `virsh` 和 libvirt domain XML。

## 理解你的 NUMA 拓扑

在进行任何调优更改之前，你必须先理解宿主机的 NUMA (非一致性内存访问) 拓扑。在多路 (multi-socket) 系统上，每个 CPU 插槽都有自己的本地内存。访问另一个插槽上的远程内存会产生更高的延迟。

如果尚未安装，请安装 `numactl`：

```bash
sudo dnf install -y numactl
```

显示你的 NUMA 拓扑：

```bash
numactl --hardware
```

双路系统上的示例输出：

```text
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
node 0 size: 65536 MB
node 0 free: 42000 MB
node 1 cpus: 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31
node 1 size: 65536 MB
node 1 free: 45000 MB
```

你也可以查看 CPU 到 NUMA 的映射关系：

```bash
lscpu --extended
```

记录哪些 CPU 核心属于哪个 NUMA 节点。在后续章节中，你将需要这些信息来进行 vCPU 绑定和内存放置。

## NUMA 系统上 `vm.min_free_kbytes` 的危险

`vm.min_free_kbytes` 内核参数控制内核为紧急分配保留的最小内存量^1^。在 NUMA 系统上，此保留量分布在所有 NUMA 节点的 lowmem 区域中，而不是作为单一的系统范围池应用。

!!! danger "将 `vm.min_free_kbytes` 设置过高可能导致立即触发 OOM (内存不足) 终止"

    在 NUMA 系统上，内核按比例将 `vm.min_free_kbytes` 保留量分布到各个区域。如果你在一个有两个 NUMA 节点的系统上将其设置为 10 GB，每个节点的 Normal 区域可能需要保留大约 5 GB 的空闲内存。如果某个 NUMA 节点的大部分内存已分配给虚拟机，它将无法满足其水位线要求。内核将该区域标记为 `all_unreclaimable` (全部不可回收) 并触发 OOM killer (内存不足终止器)，即使其他 NUMA 节点有大量空闲内存。

### 水位线分布的工作原理

内核根据 `vm.min_free_kbytes` 计算每个区域的最小水位线。你可以查看当前的水位线：

```bash
cat /proc/zoneinfo | grep -E "Node|min|low|high|managed"
```

显示每个区域分布的示例输出：

```text
Node 0, zone   Normal
  min      131072
  low      163840
  high     196608
  managed  16777216
Node 1, zone   Normal
  min      131072
  low      163840
  high     196608
  managed  16777216
```

### KVM 宿主机的安全指南

在大多数物理内存已分配给虚拟机的 KVM 宿主机上，请遵循以下指南：

- 将 `vm.min_free_kbytes` 保持在一个保守的值。一个合理的起点是总物理内存的 1-2%，不超过几百 MB。
- 检查当前值：

```bash
sysctl vm.min_free_kbytes
```

- 设置一个安全的值（以 128 GB 宿主机为例）：

```bash
sudo sysctl -w vm.min_free_kbytes=524288
```

- 使其持久化：

```bash
echo "vm.min_free_kbytes = 524288" | sudo tee /etc/sysctl.d/99-kvm-min-free.conf
sudo sysctl --system
```

!!! warning "在更改此值之前务必检查 NUMA 区域空闲内存"

    运行 `numactl --hardware` 并验证每个 NUMA 节点的空闲内存显著多于其提议的 `vm.min_free_kbytes` 值的份额。如果 NUMA 节点 0 只有 2 GB 空闲内存，而你将 `vm.min_free_kbytes` 设置为 4 GB，节点 0 将立即进入回收螺旋 (reclaim spiral)。

## 使用 libvirt 进行 vCPU 绑定

默认情况下，KVM 调度器可以将客户虚拟机的 vCPU 迁移到任何宿主机 CPU 核心。这会导致缓存抖动 (cache thrashing)、跨 NUMA 内存访问和不可预测的延迟。vCPU 绑定将每个客户虚拟机 vCPU 分配到一个特定的宿主机 CPU，以消除这些问题。

### 识别可用的宿主机 CPU

首先，确定哪些宿主机 CPU 可用以及它们的 NUMA 节点归属：

```bash
virsh capabilities | grep -A 20 "topology"
```

或使用 `lscpu`：

```bash
lscpu --extended
```

### 在 domain XML 中配置 vCPU 绑定

使用 `virsh edit` 编辑你的虚拟机 domain XML：

```bash
virsh edit your-vm-name
```

添加一个 `<cputune>` 部分^2^。每个 `<vcpupin>` 指令将一个客户 vCPU 映射到一个特定的宿主机 CPU：

```xml
<cputune>
  <vcpupin vcpu='0' cpuset='4'/>
  <vcpupin vcpu='1' cpuset='5'/>
  <vcpupin vcpu='2' cpuset='6'/>
  <vcpupin vcpu='3' cpuset='7'/>
  <emulatorpin cpuset='0-3'/>
</cputune>
```

!!! note "术语"

    - `vcpupin`：将客户 vCPU 绑定到特定的宿主机 CPU 核心。
    - `emulatorpin`：将 QEMU 仿真器线程（负责 I/O、定时器和设备仿真）绑定到特定的宿主机 CPU。

### `emulatorpin` 的重要性

`<emulatorpin>` 指令至关重要，但经常被忽视。没有它，QEMU 仿真器线程可以自由运行在任何宿主机 CPU 上，包括你为客户虚拟机 vCPU 隔离的 CPU。这会导致争用，可能使客户虚拟机的 vCPU 被取消调度，导致时钟漂移和应用程序超时。

!!! danger "始终配置 `emulatorpin`"

    没有 `emulatorpin`，QEMU 仿真器线程可以在你隔离的 vCPU 核心上运行，导致 hypervisor 取消调度客户虚拟机 CPU。对时序敏感的应用程序（如 F5 BIG-IP、数据库集群和实时工作负载）在 vCPU 失去调度时间时将出现故障。

将仿真器线程绑定到你的维护 (housekeeping) CPU（未分配给任何客户 vCPU 的核心）：

```xml
<emulatorpin cpuset='0-3'/>
```

### 绑定规则

遵循以下步骤进行有效的 vCPU 绑定：

- 将所有 vCPU 绑定到与虚拟机内存分配相同的 NUMA 节点。
- 使用专用的 1:1 绑定，其中每个客户 vCPU 映射到恰好一个宿主机 CPU。避免使用多个 vCPU 共享一个 cpuset 的浮动池。
- 为宿主机内核、IRQ 处理、QEMU 仿真器线程和系统守护进程保留维护 CPU（通常是核心 0-3）。
- 永远不要将维护核心与客户 vCPU 绑定重叠。允许客户 vCPU 池包含维护核心会导致调度争用。

### 完整的 vCPU 绑定示例

此示例展示了一个 12 个 vCPU 的虚拟机绑定到 NUMA 节点 0，仿真器线程在维护核心上：

```xml
<vcpu placement='static'>12</vcpu>
<cputune>
  <vcpupin vcpu='0' cpuset='4'/>
  <vcpupin vcpu='1' cpuset='5'/>
  <vcpupin vcpu='2' cpuset='6'/>
  <vcpupin vcpu='3' cpuset='7'/>
  <vcpupin vcpu='4' cpuset='8'/>
  <vcpupin vcpu='5' cpuset='9'/>
  <vcpupin vcpu='6' cpuset='10'/>
  <vcpupin vcpu='7' cpuset='11'/>
  <vcpupin vcpu='8' cpuset='12'/>
  <vcpupin vcpu='9' cpuset='13'/>
  <vcpupin vcpu='10' cpuset='14'/>
  <vcpupin vcpu='11' cpuset='15'/>
  <emulatorpin cpuset='0-3'/>
</cputune>
<numatune>
  <memory mode='strict' nodeset='0'/>
</numatune>
```

启动虚拟机后验证绑定：

```bash
virsh vcpuinfo your-vm-name
```

## 使用 `isolcpus` 进行 CPU 隔离

单独的 vCPU 绑定并不能防止宿主机内核调度器将其他进程放置到这些 CPU 上。要完全将 CPU 专用于虚拟机，需使用内核 CPU 隔离。

### 内核命令行参数

将以下内容添加到内核命令行，将非维护 CPU（在此示例中为 CPU 4-63）隔离以专供虚拟机使用：

```bash
sudo grubby --args="isolcpus=4-63 nohz_full=4-63 rcu_nocbs=4-63" --update-kernel=ALL
```

重启以应用：

```bash
sudo reboot
```

这些参数协同工作^3^：

- `isolcpus=4-63`：将 CPU 4-63 从通用调度器中移除。只有明确绑定的任务才能在其上运行。
- `nohz_full=4-63`：当只有一个任务在隔离 CPU 上运行时，禁用调度器时钟节拍，减少抖动^4^。
- `rcu_nocbs=4-63`：将 RCU (读-复制-更新) 回调处理从隔离 CPU 卸载到专用的 kthreads，然后这些 kthreads 可以被亲和到维护 CPU。

### 验证 CPU 隔离

重启后，确认隔离已激活：

```bash
cat /sys/devices/system/cpu/isolated
```

预期输出：

```text
4-63
```

你可以使用压力测试进一步验证。运行 `stress-ng` 并确认进程仅落在维护 CPU 上：

```bash
taskset -c 0-63 stress-ng --cpu 8 --timeout 30s &
watch -n1 'ps -eo pid,psr,comm | grep stress'
```

`PSR` 列应该只显示 CPU 0-3（非隔离的维护核心）。如果 stress 进程出现在隔离 CPU 上，则隔离配置需要进行故障排除。

## 虚拟化主机的 `tuned` 配置文件

`tuned` 守护进程提供系统调优配置文件，自动化内核参数和 CPU 调控器设置。Rocky Linux 包含专为虚拟化主机设计的配置文件。

### 安装虚拟化配置文件

```bash
sudo dnf install -y tuned-profiles-cpu-partitioning
```

### 可用的配置文件

查看所有可用的配置文件：

```bash
tuned-adm list
```

KVM 主机的相关配置文件有：

- `virtual-host`：虚拟化主机的基本调优。启用透明大页并调整内核调度参数。
- `cpu-partitioning`：结合 CPU 隔离与优化调度的高级配置文件。最适合对延迟敏感的虚拟机工作负载。

### 配置 `cpu-partitioning`

`cpu-partitioning` 配置文件需要在激活前进行配置。编辑变量文件：

```bash
sudo vi /etc/tuned/cpu-partitioning-variables.conf
```

将隔离和不平衡的核心设置为与你的 CPU 隔离计划匹配：

```ini
isolated_cores=4-63
no_balance_cores=4-63
```

激活配置文件：

```bash
sudo tuned-adm profile cpu-partitioning
```

验证活动的配置文件：

```bash
tuned-adm active
```

预期输出：

```text
Current active profile: cpu-partitioning
```

激活配置文件后，重启以确保所有内核参数完全应用：

```bash
sudo reboot
```

### 验证 `tuned` 设置

重启后，确认配置文件已应用预期的内核参数：

```bash
cat /proc/cmdline | tr ' ' '\n' | grep -E "isolcpus|nohz_full|rcu_nocbs"
```

## 使用 `virsh` 进行 NUMA 感知的虚拟机放置

为了获得最佳性能，虚拟机的内存和 vCPU 必须在同一个 NUMA 节点上。访问远程 NUMA 节点上的内存会增加显著的延迟。

### 配置 NUMA 内存绑定

在虚拟机 domain XML 中，使用 `<numatune>` 元素将虚拟机内存绑定到特定的 NUMA 节点^2^：

```xml
<numatune>
  <memory mode='strict' nodeset='0'/>
</numatune>
```

`strict` 模式确保此虚拟机的所有内存分配都来自 NUMA 节点 0。如果该节点没有足够的空闲内存，分配将失败，不会静默地回退到远程内存。这可能会触发 OOM killer 而不是阻止虚拟机启动，因此请确保目标 NUMA 节点在启动虚拟机之前有足够的空闲内存。

### 将 vCPU 绑定与 NUMA 内存对齐

确保你绑定的 vCPU 所属的 CPU 核心与内存绑定在同一个 NUMA 节点上。使用之前的 `numactl --hardware` 输出，如果 NUMA 节点 0 包含 CPU 0-15，则将你的 vCPU 绑定到该范围内的核心（排除维护核心）：

```xml
<vcpu placement='static'>8</vcpu>
<cputune>
  <vcpupin vcpu='0' cpuset='4'/>
  <vcpupin vcpu='1' cpuset='5'/>
  <vcpupin vcpu='2' cpuset='6'/>
  <vcpupin vcpu='3' cpuset='7'/>
  <vcpupin vcpu='4' cpuset='8'/>
  <vcpupin vcpu='5' cpuset='9'/>
  <vcpupin vcpu='6' cpuset='10'/>
  <vcpupin vcpu='7' cpuset='11'/>
  <emulatorpin cpuset='0-3'/>
</cputune>
<numatune>
  <memory mode='strict' nodeset='0'/>
</numatune>
```

### 运行时验证 NUMA 放置

启动虚拟机后，验证其 NUMA 内存使用情况：

```bash
virsh numatune your-vm-name
```

检查虚拟机进程正在使用哪些 NUMA 节点：

```bash
numastat -c qemu-kvm
```

### 在 NUMA 节点之间重新平衡虚拟机

如果一个 NUMA 节点过度使用而另一个利用不足，在节点之间重新分配虚拟机。关闭虚拟机，更新其 `<numatune>` 和 `<cputune>` 以指向另一个 NUMA 节点，然后重新启动：

```bash
virsh shutdown your-vm-name
virsh edit your-vm-name
# 将 nodeset 和 vcpupin 更改为指向 NUMA 节点 1 的 CPU
virsh start your-vm-name
```

## KVM 的大页内存 (Hugepages)

大页通过使用更大的内存页面（2 MB 或 1 GB，而不是默认的 4 KB）来减少 TLB (转译后备缓冲区) 未命中。这显著改善了虚拟机的内存访问性能。

### 配置 2 MB 大页

计算所需的大页数量。对于一个 16 GB 内存的虚拟机：

```text
16384 MB / 2 MB = 8192 个大页
```

保留大页：

```bash
echo "vm.nr_hugepages = 8192" | sudo tee /etc/sysctl.d/99-hugepages.conf
sudo sysctl --system
```

验证分配：

```bash
grep HugePages /proc/meminfo
```

### 在虚拟机 domain XML 中配置大页

添加 `<memoryBacking>` 元素以使用大页^2^：

```xml
<memoryBacking>
  <hugepages>
    <page size='2048' unit='KiB'/>
  </hugepages>
  <locked/>
</memoryBacking>
```

!!! note "`locked` 元素"

    `<locked/>` 元素防止宿主机交换虚拟机内存。这对于性能是推荐的，但需要足够的物理内存。如果使用 `<locked/>`，请确保为所有虚拟机分配了足够的大页。

### NUMA 感知的大页分配

对于 NUMA 系统，按节点分配大页以确保它们在需要的地方可用：

```bash
echo 4096 | sudo tee /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
echo 4096 | sudo tee /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages
```

将大页分配与每个 NUMA 节点上的虚拟机放置相匹配。

## 综合配置

一个完全调优的虚拟机配置结合了本指南中涵盖的所有元素^5^。以下是一个双路 Rocky Linux 9 宿主机上性能关键型虚拟机的完整示例：

```xml
<domain type='kvm'>
  <name>production-vm</name>
  <memory unit='GiB'>16</memory>
  <vcpu placement='static'>8</vcpu>

  <cpu mode='host-passthrough'>
    <topology sockets='1' dies='1' cores='8' threads='1'/>
    <numa>
      <cell id='0' cpus='0-7' memory='16' unit='GiB'/>
    </numa>
  </cpu>

  <cputune>
    <vcpupin vcpu='0' cpuset='4'/>
    <vcpupin vcpu='1' cpuset='5'/>
    <vcpupin vcpu='2' cpuset='6'/>
    <vcpupin vcpu='3' cpuset='7'/>
    <vcpupin vcpu='4' cpuset='8'/>
    <vcpupin vcpu='5' cpuset='9'/>
    <vcpupin vcpu='6' cpuset='10'/>
    <vcpupin vcpu='7' cpuset='11'/>
    <emulatorpin cpuset='0-3'/>
  </cputune>

  <numatune>
    <memory mode='strict' nodeset='0'/>
  </numatune>

  <memoryBacking>
    <hugepages>
      <page size='2048' unit='KiB'/>
    </hugepages>
    <locked/>
  </memoryBacking>

  <!-- 剩余的 domain 配置（磁盘、网络等） -->
</domain>
```

## 结论

在 Rocky Linux 上对 KVM/libvirt 进行性能调优需要一种整体性的方法，同时处理 CPU 调度、内存放置和 I/O 配置。关键原则是：

- 在进行任何更改之前理解你的 NUMA 拓扑。
- 将 vCPU 绑定到与虚拟机内存位于同一 NUMA 节点上的特定核心。
- 始终配置 `emulatorpin` 以防止 QEMU 线程与客户虚拟机 vCPU 争用。
- 对延迟敏感的工作负载使用 `isolcpus`、`nohz_full` 和 `rcu_nocbs` 隔离 CPU。
- 使用 `cpu-partitioning` tuned 配置文件进行一致的系统范围调优。
- 在 NUMA 系统上谨慎处理 `vm.min_free_kbytes`，以避免 OOM 情况。

## 参考文献

1. "Documentation for /proc/sys/vm/" by the Linux kernel documentation project [https://docs.kernel.org/admin-guide/sysctl/vm.html](https://docs.kernel.org/admin-guide/sysctl/vm.html)
2. "Domain XML format" by the libvirt project [https://libvirt.org/formatdomain.html](https://libvirt.org/formatdomain.html)
3. "The kernel's command-line parameters" by the Linux kernel documentation project [https://docs.kernel.org/admin-guide/kernel-parameters.html](https://docs.kernel.org/admin-guide/kernel-parameters.html)
4. "NO_HZ: Reducing scheduling-clock ticks" by the Linux kernel documentation project [https://docs.kernel.org/timers/no_hz.html](https://docs.kernel.org/timers/no_hz.html)
5. "KVM real time guest configuration" by the libvirt project [https://libvirt.org/kbase/kvm-realtime.html](https://libvirt.org/kbase/kvm-realtime.html)
