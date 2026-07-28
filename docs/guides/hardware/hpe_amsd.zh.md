---
title: HPE ProLiant 无代理管理服务
author: Neel Chauhan
contributors: Ganna Zhyrnova
tested_with: 9.3
tags:
  - hardware
---

# HPE ProLiant 无代理管理服务

## 简介

HPE ProLiant 服务器配套有一款名为无代理管理服务（Agentless Management Service）的软件，按照 HPE 的说法：

> 使用带外通信来提高安全性和稳定性。

此外：

> 借助无代理管理，健康监控和告警功能已内置于系统中，在服务器接通辅助电源后即可开始工作。

例如，在作者的家庭实验室中，该服务用于降低 HPE ProLiant ML110 Gen11 的风扇转速。

## 前置条件与假设

使用本操作流程的最低要求如下：

* 一台 HP/HPE ProLiant Gen8 或更新的服务器，已启用 iLO（集成无人值守管理）并在网络中可见

## 安装 `amsd`

要安装 `amsd`，首先需要安装 EPEL（企业 Linux 额外软件包）并运行更新：

```bash
dnf -y install epel-release && dnf -y update
```

然后将以下内容添加到 `/etc/yum.repos.d/spp.repo`：

```bash

[spp]
name=Service Pack for ProLiant
baseurl=https://downloads.linux.hpe.com/repo/spp-gen11/redhat/9/x86_64/current
enabled=1
gpgcheck=1
gpgkey=https://downloads.linux.hpe.com/repo/spp/GPG-KEY-spp 
```

将 `9` 替换为 Rocky Linux 的主版本号，将 `gen11` 替换为服务器的代数。作者使用的是 ML110 Gen11，如果使用的是 DL360 Gen10，则应使用 `gen10`。

接着，安装并启用 `amsd`：

```bash
dnf -y update && dnf -y install amsd
systemctl enable --now amsd
```

如果要检查 `amsd` 是否正常工作，可以通过 Web 浏览器登录 iLO。如果安装正确，iLO 应报告服务器正在运行 Rocky Linux：

![HPE iLO 显示 Rocky Linux 9.3](../images/hpe_ilo_amsd.png)

## 总结

对 HPE 服务器的一个常见批评是，当使用第三方组件（如非 HPE 官方认可的 SSD 或其他附加 PCI Express 卡，例如视频采集卡）时，风扇转速会很高。即使只使用 HPE 品牌的组件，使用 `amsd` 也能让 HPE ProLiant 服务器比仅使用 Rocky Linux 时运行更凉爽、更安静。
