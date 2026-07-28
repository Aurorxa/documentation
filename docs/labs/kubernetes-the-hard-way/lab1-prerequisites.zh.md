---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 1: 前置条件

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将检查遵循本教程所需的机器要求。

## 虚拟机或物理机

本教程需要四台运行 Rocky Linux 9.5 的 x86_64 虚拟机或物理机（Incus 或 LXD 容器也应可行）。下表列出了这四台机器及其 CPU、内存和存储要求。

| 名称    | 描述            | CPU | RAM   | 存储 |
|---------|----------------|-----|-------|------|
| jumpbox | 管理主机        | 1   | 512MB | 10GB |
| server  | Kubernetes 服务器 | 1   | 2GB   | 20GB |
| node-0  | Kubernetes worker 节点 | 1   | 2GB   | 20GB |
| node-1  | Kubernetes worker 节点 | 1   | 2GB   | 20GB |

如何配置这些机器由你决定；唯一的要求是每台机器满足上述系统要求，包括机器规格和操作系统版本。一旦四台机器全部就绪，在每台机器上运行 `uname` 命令来验证系统要求：

```bash
uname -mov
```

运行 `uname` 命令后，应看到以下输出：

```text
#1 SMP PREEMPT_DYNAMIC Wed Feb 19 16:28:19 UTC 2025 x86_64 GNU/Linux
```

输出中的 `x86_64` 确认系统是 x86_64 架构。这对各种基于 AMD 和 Intel 的系统都应该是如此。

下一步: [搭建 jumpbox](lab2-jumpbox.md)
