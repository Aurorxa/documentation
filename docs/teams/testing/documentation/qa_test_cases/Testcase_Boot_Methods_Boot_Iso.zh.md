---
title: QA:Testcase 引导方法 Boot Iso
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

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#initialization-requirements](../../guidelines/release_criteria/r9/9_release_criteria.md#initialization-requirements) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试验证从 Rocky Linux boot.iso 引导时 Anaconda 安装程序是否能正确启动。

## 设置

1. 准备你的系统以引导 boot.iso 镜像。这可能涉及将镜像写入 USB 密钥或刻录到光盘。此外，将 boot.iso 作为虚拟光盘附加到虚拟机实例，或通过基板管理控制器虚拟介质挂载将 boot.iso 挂载到服务器应该是可行的，但并非明确要求。

## 如何测试

1. 从准备好的光盘、USB 介质或虚拟设备附件启动系统。
2. 在引导菜单中选择适当的选项以启动安装程序。

## 预期结果

1. 显示图形引导菜单，供用户选择安装选项。菜单导航和条目选择必须正常工作。如果没有选择选项，安装程序应在合理的超时后加载。
2. 系统引导进入 Anaconda 安装程序。

{% include 'teams/testing/qa_testcase_bottom.md' %}
