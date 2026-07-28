---
title: QA:Testcase 固件 RAID
author: Al Bowles
contributors:
tested_with: 8.10, 9.7
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

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#Firmware RAID](../../guidelines/release_criteria/r9/9_release_criteria.md#firmware-raid) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

安装程序必须能够检测固件 RAID（Firmware RAID）设备并安装到其上。注意，特定于系统的错误不计为阻塞问题。某些硬件支持可能损坏或完全不可用。此标准不考虑 DUD（Driver Update Disk，驱动更新磁盘）。

## 设置

1. 添加此测试用例的设置步骤。

## 如何测试

1. 首先执行此操作...
2. 然后执行此操作...

## 预期结果

1. 这是你应该看到/验证的内容。
2. 你还应该看到/验证此内容。

{% include 'teams/testing/qa_testcase_bottom.md' %}
