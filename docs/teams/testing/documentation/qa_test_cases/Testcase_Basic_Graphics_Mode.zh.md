---
title: QA:Testcase 基本图形模式
author: Trevor Cooper
contributors:
tested_with: 8.10
tags:
  - testing
  - qa
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
  ver: 8
  level: Final
render_macros: true
---

!!! error "关联的发布标准过于笼统且不可测试"
    此测试用例的关联发布标准 [Release_Criteria#basic-graphics-mode-behaviors](../../guidelines/release_criteria/r8/8_release_criteria.md#basic-graphics-mode-behaviors) 过于笼统，**必须**修改为足够具体以便可测试。

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#basic-graphics-mode-behaviors](../../guidelines/release_criteria/r8/8_release_criteria.md#basic-graphics-mode-behaviors) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例将验证阻止发布的安装程序（release-blocking installers）在受支持的系统和硬件类别上使用通用视频驱动程序选项（"基本图形模式"）是否按预期运行。

{% include 'teams/testing/qa_testcase_supported_systems.md' %}

## 设置

1. 获取对要安装的受支持系统和硬件类别的访问权限。
2. 为要测试的所选 ISO 准备适当的介质。
    - 示例：[QA:Testcase Media USB dd](Testcase_Media_USB_dd.md)

## 如何测试

1. 从准备好的光盘、USB 介质或虚拟设备附件启动系统。
    - 示例：[QA:Testcase Boot Methods Boot ISO](Testcase_Boot_Methods_Boot_Iso.md)、[QA:Testcase Boot Methods DVD](Testcase_Boot_Methods_Dvd.md)
2. 在引导菜单中选择适当的选项以启动安装程序。
3. 在安装程序中选择适当的选项以在基本图形模式下进行安装。
4. 在测试系统上继续安装。**注意：根据安装程序的选择，这可能会销毁测试系统上的所有数据。**

!!! error "数据丢失"
    如果你选择完成测试系统的安装，系统上的任何/所有数据可能会丢失。请不要在你需要保留内容的系统上进行安装。

## 预期结果

1. 可以在 Anaconda 安装程序中选择基本图形模式。
2. Anaconda 安装程序呈现一个可用的图形安装环境。
3. 被测系统可以正常安装。
4. 重启后系统进入图形环境。
5. 登录后用户能够操作图形环境。

{% include 'teams/testing/qa_testcase_bottom.md' %}
