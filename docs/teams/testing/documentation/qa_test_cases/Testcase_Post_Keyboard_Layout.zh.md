---
title: QA:Testcase 键盘布局
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
    此测试用例关联 [Release_Criteria#keyboard-layout](../../guidelines/release_criteria/r9/9_release_criteria.md#keyboard-layout) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

由于存在大量不同的键盘布局，有必要测试键盘功能在整个系统中是否正常工作。

## 准备工作

- 获取几种不同系统配置的访问权限，尤其是有 UI 和无 UI 的配置，以及带磁盘加密的配置。
- 获取物理机或虚拟机主机的访问权限，以便安装新机器
    - 为要测试的 ISO 准备适当的介质。
    - 示例：[QA:Testcase Media USB dd](Testcase_Media_USB_dd.md)

## 测试方法

### 安装程序

1. 启动安装程序
2. 选择一种语言
3. 确认键盘布局是否与语言设置正确对应
4. 如有需要，更改键盘布局进行测试
5. 在 Anaconda 各处输入文字，确认所选键盘布局正常工作

### 磁盘加密

1. 配置带有磁盘加密的系统
2. 检查在图形 UI 启动时磁盘加密密码是否正常工作
3. 检查在文本模式启动时磁盘加密密码是否正常工作

### 文本模式

检查所选的键盘布局在文本模式下是否正常工作。

### GNOME 和应用程序

1. 检查登录时键盘布局在图形 UI 登录屏幕上是否正常工作
2. 检查 GNOME UI 在所选键盘布局下是否正常工作
3. 最后检查一些应用程序的键盘是否按预期工作

## 预期结果

整个过程中的测试均能成功完成。

{% include 'teams/testing/qa_testcase_bottom.md' %}
