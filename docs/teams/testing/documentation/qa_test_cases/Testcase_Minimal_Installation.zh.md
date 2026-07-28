---
title: QA:Testcase 最小化安装
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
    此测试用例关联 [Release_Criteria#Minimal Installation](../../guidelines/release_criteria/r9/9_release_criteria.md#minimal-installation) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例验证网络化最小化安装能够安装 'Minimal'（最小化）软件包集。安装不应需要使用本地软件包来完成。

{% include 'teams/testing/qa_data_loss_warning.md' %}

## 设置

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

## 如何测试

1. 从 Installation Source（安装源）spoke 中，配置一个来自 [MirrorManager](https://mirrors.rockylinux.org) 的、适合被测架构的远程仓库源。
1. 从 Software Selection（软件选择）spoke 中，选择 Minimal（最小化）软件包集。
1. 使用所需参数完成安装。

## 预期结果

1. 安装应完成并成功启动。

## 在 openQA 中测试

以下 openQA 测试套件满足此发布标准：

- `install_minimal`

{% include 'teams/testing/qa_testcase_bottom.md' %}
