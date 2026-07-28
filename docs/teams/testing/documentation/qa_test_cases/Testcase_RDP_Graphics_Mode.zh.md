---
title: QA:Testcase RDP 图形模式
author: Trevor Cooper
contributors:
tested_with: 8.10, 9.7, 10.1
tags:
  - testing
  - qa
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
  ver: 10
  level: Final
render_macros: true
---

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#rdp-graphics-mode-behaviors](../../guidelines/release_criteria/r10/10_release_criteria.md#rdp-graphics-mode-behaviors) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例将验证发布阻塞级别的安装程序在支持的系统和硬件类别上使用 RDP 安装方法（通过 gnome-remote-desktop）是否按预期工作。

{% include 'teams/testing/qa_testcase_supported_systems.md' %}

## 准备工作

1. 获取要安装的受支持系统和硬件类别的访问权限。
2. 为要测试的 ISO 准备适当的介质。
    - 示例：[QA:Testcase Media USB dd](Testcase_Media_USB_dd.md)
3. 获取安装有 RDP 客户端的远程系统访问权限，用于 RDP 连接。

!!! info "推荐的 RDP 客户端"
    [`freerdp`](https://freerdp.com) 是 Rocky Linux 中提供的 RDP 客户端，但任何 RDP 客户端均可使用。

## 测试方法

1. 从准备好的光盘、USB 介质或虚拟设备附件启动系统。
    - 示例：[QA:Testcase Custom Boot Methods Boot ISO](Testcase_Custom_Boot_Methods_Boot_Iso.md)
2. 中断内核启动，在启动命令行中指定适当的 RDP 安装选项。
3. 继续在测试系统上进行安装。**根据安装程序的选择，这可能会销毁测试系统上的所有数据。**
4. 根据 `direct` 或 `connect` 模式的选择，向内连接到被测系统或等待它连接到您正在监听的 RDP 客户端。

!!! error "数据丢失"
    如果您选择完成测试系统的安装，系统上的任何/所有数据可能会丢失。请不要在需要保留内容的系统上安装。

## 预期结果

1. 可以通过 RDP 连接（direct 模式）或接收（connect 模式）Anaconda 安装程序。
2. Anaconda 安装程序呈现可用的图形安装环境。
3. 被测系统可以正常安装。
4. 重启后系统启动到图形环境。
5. 登录后用户可以操作图形环境。

{% include 'teams/testing/qa_testcase_bottom.md' %}
