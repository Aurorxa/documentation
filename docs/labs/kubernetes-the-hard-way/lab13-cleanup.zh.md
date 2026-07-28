---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - kubernetes
  - k8s
  - lab exercise
---

# 实验 13: 清理

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

在本实验中，你将删除在本教程期间创建的计算资源。

## 计算实例

本指南的早期版本使用了 GCP 资源来处理计算和网络的各个方面。当前版本与平台无关；所有配置都在 `jumpbox`、`server` 或节点上执行。

清理操作非常简单，只需删除你为本练习创建的所有虚拟机即可。

下一步: [重新开始](lab0-README.md)
