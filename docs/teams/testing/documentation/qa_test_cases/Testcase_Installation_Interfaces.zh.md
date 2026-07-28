---
title: QA:Testcase 安装界面
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
    此测试用例关联 [Release_Criteria#Installation Interfaces](../../guidelines/release_criteria/r9/9_release_criteria.md#installation-interfaces) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例验证安装程序能够使用所有 Anaconda spoke 完成安装。

{% include 'teams/testing/qa_data_loss_warning.md' %}

## 设置

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

## 如何测试
<!-- 本地化 -->
1. 在 Keyboard Layout（键盘布局）spoke 中选择键盘布局
1. 在 Language（语言）spoke 中设置语言支持
1. 在 Time and Date（时间和日期）spoke 中设置系统时间和日期
<!-- 用户设置 -->
1. 在 Root Password（root 密码）spoke 中设置 root 密码
1. 在用户创建 spoke 中创建用户
<!-- 软件 -->
1. 从 Installation Source（安装源）spoke 中选择安装源
1. 从 Package Selection（软件包选择）spoke 中选择要安装的软件包集
<!-- 系统 -->
1. 在 Installation Destination（安装目标）spoke 中设置操作系统应安装到的磁盘
1. 从 Kdump spoke 中设置 kdump 状态
1. 从 Network and Hostname（网络和主机名）spoke 中配置系统的网络和主机名
1. 从 Security Policy（安全策略）spoke 中选择安全策略

## 预期结果

1. 安装应完成并成功启动。

## 在 openQA 中测试

以下 openQA 测试套件满足此发布标准：

- `install_arabic_language` 或 `install_asian_language`
<!--
TODO
- 包含 kdump 的某些测试
- 设置主机名
- 设置安全策略
-->

{% include 'teams/testing/qa_testcase_bottom.md' %}
