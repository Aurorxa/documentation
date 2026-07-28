---
title: 0. cloud-init
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - cloud-init
---

## Rocky Linux 上的 cloud-init 指南

欢迎阅读 Rocky Linux 上 `cloud-init` 的全面指南。本系列从云实例初始化的基本概念开始，逐步深入到高级的、真实世界的配置和故障排除技术。无论你是正在设置第一个云服务器的新用户，还是正在构建自定义镜像的经验丰富的管理员，本指南都能为你提供有价值的内容。

要充分利用各章节，你应该按顺序阅读，以前面章节的知识为基础。

---

## 本指南的章节

**[1. 基础知识](./01_fundamentals.md)**
> 了解 `cloud-init` 是什么，为什么它对云计算至关重要，以及其执行生命周期中的各个阶段。

**[2. 初次接触](./02_first_contact.md)**
> 你的第一个动手练习。使用基本的 `user-data` 文件启动一个云镜像并执行简单的定制。

**[3. 配置引擎](./03_configuration_engine.md)**
> 深入 `cloud-init` 模块系统。学习如何使用最重要的模块来管理用户、软件包和文件。

**[4. 高级配置](./04_advanced_provisioning.md)**
> 处理复杂场景，包括如何定义静态网络配置，以及如何将脚本和 cloud-configs 组合为单个负载。

**[5. 镜像构建者的视角](./05_image_builders_perspective.md)**
> 将你的视角转变为镜像构建者。学习如何创建带有预制默认值的"黄金镜像"，以及如何正确地通用化它们以供克隆。

**[6. 故障排除](./06_troubleshooting.md)**
> 学习 `cloud-init` 取证的基本技巧。理解日志、状态命令和常见陷阱，以有效地诊断和解决问题。

**[7. 为 cloud-init 做贡献](./07_contributing.md)**
> 超越用户身份。本章提供了一张地图，让你理解 `cloud-init` 的源代码，并向这个开源项目做出你的首次贡献。
