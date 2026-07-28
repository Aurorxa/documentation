---
title: QA:Testcase 引导加载程序磁盘选择
author: Al Bowles
contributors: Trevor Cooper
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
    此测试用例关联 [Release_Criteria#Bootloader Disk Selection](../../guidelines/release_criteria/r9/9_release_criteria.md#bootloader-disk-selection) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例验证用户能够选择一个备用磁盘来安装引导加载程序（bootloader）。它还验证，如果用户有此意愿，他们可以选择不安装引导加载程序。

{% include 'teams/testing/qa_data_loss_warning.md' %}

## 设置

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

## 如何测试

1. 在 Installation Destination（安装目标）spoke 中，选择要安装的磁盘，然后点击屏幕底部的 "Full disk summary and bootloader..."（完整磁盘摘要和引导加载程序...）按钮：![Full disk summary and bootloader...](../../../../assets/teams/testing/bootloader.png){ loading=lazy }
1. 点击所需安装引导加载程序的磁盘旁边的复选框
1. 或者，取消所有磁盘旁边的引导复选框以跳过引导加载程序安装
1. 在测试系统上继续安装。

## 预期结果

1. 安装应成功完成。
1. 注意，如果没有安装引导加载程序，系统在安装完成后可能无法引导。这符合预期。

{% include 'teams/testing/qa_testcase_bottom.md' %}
