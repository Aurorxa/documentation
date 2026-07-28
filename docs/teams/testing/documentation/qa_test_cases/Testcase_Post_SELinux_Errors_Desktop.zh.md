---
title: QA:Testcase 桌面客户端上的 SELinux 错误
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
    此测试用例关联 [Release_Criteria#selinux-and-crash-notifications-desktop-only](../../guidelines/release_criteria/r9/9_release_criteria.md#selinux-and-crash-notifications-desktop-only) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

本质上是运行 Workstation 或 Graphical Server 安装较长时间，在使用过程中检查是否出现任何 SELinux 审计 (audit) 消息。

## 准备工作

获取一个装有 Workstation 或 Graphical Server 安装的合适系统。

## 测试方法

1. 设置新机器或获取已安装机器的访问权限
2. 在系统和各种应用程序中进行点选操作，模拟用户行为
3. 让系统再运行几分钟，如果可能的话运行几小时

## 预期结果

1. 打开 SETroubleshoot 应用程序并调用错误摘要。
2. 不应显示任何错误或拒绝 (denial) 信息

{% include 'teams/testing/qa_testcase_bottom.md' %}
