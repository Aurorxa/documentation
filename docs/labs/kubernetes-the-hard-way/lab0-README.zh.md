---
title: 引言
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
---

# Kubernetes The Hard Way (Rocky Linux)

!!! info

    这是原始 ["Kubernetes the hard way"](https://github.com/kelseyhightower/kubernetes-the-hard-way) 的一个分支，原作者为 Kelsey Hightower (GitHub: kelseyhightower)。与基于 Debian 类发行版、面向 ARM64 架构的原始版本不同，本分支面向企业级 Linux 发行版，如运行在 x86_64 架构上的 Rocky Linux。

本教程将引导你通过"硬核方式"搭建 Kubernetes。它不适合那些寻找全自动化工具来搭建 Kubernetes 集群的人。Kubernetes The Hard Way 专为学习而设计，意味着你要走更长的路，以确保理解引导 Kubernetes 集群所需的每个任务。

请不要将本教程的结果视为可用于生产环境，社区也可能不会提供支持，但不要让这些阻碍你学习！

## 版权

![Creative Commons License](images/cc_by_sa.png)

本作品的许可协议为 [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/)。

## 目标受众

本教程的目标受众是任何希望理解 Kubernetes 基本原理以及核心组件如何协同工作的人。

## 集群详情

Kubernetes The Hard Way 将引导你引导一个基本的 Kubernetes 集群，所有控制平面组件运行在单个节点上，外加两个 worker 节点，这足以学习核心概念。

组件版本：

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.32.x
* [containerd](https://github.com/containerd/containerd) v2.0.x
* [cni](https://github.com/containernetworking/cni) v1.6.x
* [etcd](https://github.com/etcd-io/etcd) v3.4.x

## 实验

本教程需要四台基于 x86_64 的虚拟机或物理机，并连接到同一网络。虽然教程使用 x86_64 架构的机器，但您可以将所学经验应用于其他平台。

* [前置条件](lab1-prerequisites.md)
* [搭建 Jumpbox](lab2-jumpbox.md)
* [配置计算资源](lab3-compute-resources.md)
* [配置 CA 并生成 TLS 证书](lab4-certificate-authority.md)
* [生成用于认证的 Kubernetes 配置文件](lab5-kubernetes-configuration-files.md)
* [生成数据加密配置和密钥](lab6-data-encryption-keys.md)
* [引导 etcd 集群](lab7-bootstrapping-etcd.md)
* [引导 Kubernetes 控制平面](lab8-bootstrapping-kubernetes-controllers.md)
* [引导 Kubernetes Worker 节点](lab9-bootstrapping-kubernetes-workers.md)
* [配置 kubectl 远程访问](lab10-configuring-kubectl.md)
* [配置 Pod 网络路由](lab11-pod-network-routes.md)
* [冒烟测试](lab12-smoke-test.md)
* [清理](lab13-cleanup.md)
