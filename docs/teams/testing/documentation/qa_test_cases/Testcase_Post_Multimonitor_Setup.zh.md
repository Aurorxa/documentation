---
title: QA:Testcase 多显示器设置
author: Lukas Magauer
contributors:
tested_with: 8.10, 9.7, 10.1
tags:
  - testing
  - qa
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
  ver:
    - 8
    - 9
  level: Final
render_macros: true
---

!!! info "发布适用范围"
    此测试用例适用于以下版本的 {{ rc.prod }}：{% for version in rc.ver %}{{ version }}{% if not loop.last %}, {% endif %}{% endfor %}

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#dual-monitor-setup-desktop-only](../../guidelines/release_criteria/r9/9_release_criteria.md#dual-monitor-setup-desktop-only) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试覆盖在多显示器设置下 GNOME 是否按预期运行的检查。

## 准备工作

您需要一台可以重新安装并连接多个屏幕的机器，或一个能够提供多个屏幕的虚拟化软件（如 VMware Workstation（[Pro](https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html) 或 [Player](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html)）或 [VMware Fusion](https://www.vmware.com/products/fusion/fusion-evaluation.html)，[VMware ESXi](https://customerconnect.vmware.com/en/web/vmware/evalcenter?p=vsphere-eval-7) 也有一个[非官方方法](https://communities.vmware.com/t5/VMware-vSphere-Discussions/ESXi-6-7-Multiple-Monitors-for-VMs/td-p/2748906)）

## 测试方法

1. 连接多个屏幕运行安装程序，并使用 Workstation 或 Graphical Server 组进行安装
2. 安装完成后登录机器

## 预期结果

在安装和使用过程中不应出现任何图形故障或缩放问题。

{% include 'teams/testing/qa_testcase_bottom.md' %}
