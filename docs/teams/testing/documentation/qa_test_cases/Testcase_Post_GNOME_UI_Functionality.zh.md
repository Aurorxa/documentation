---
title: QA:Testcase GNOME UI 功能
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

!!! error "引用的发布标准过于笼统且无法测试"
    此测试用例关联的发布标准 [Release_Criteria#default-panel-functionality-desktop-only](../../guidelines/release_criteria/r9/9_release_criteria.md#default-panel-functionality-desktop-only) 过于笼统，**必须**修改到足够具体以便可测试。

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#default-panel-functionality-desktop-only](../../guidelines/release_criteria/r9/9_release_criteria.md#default-panel-functionality-desktop-only) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试集负责验证 GNOME UI 的正确功能。

## 准备工作

获取一个装有 Workstation 或 Graphical Server 安装的合适系统。

## 测试方法

1. 通过 UI 登录 Rocky 机器
2. 浏览 GNOME UI

## 预期结果

1. 登录后应进入桌面，背景和 GNOME 顶栏可见
2. 点击右上角的"活动"按钮应弹出概览界面
3. 左侧应有常用应用程序侧边栏
4. 点击九点图标后应显示所有应用程序
5. 返回桌面检查右上角和中间的系统与时钟面板功能

在 GNOME UI 中进行一些基础的点选操作也是很好的实践，例如打开应用程序、与应用程序标题栏交互、将应用程序移动到不同桌面或在系统设置中更改设置。

{% include 'teams/testing/qa_testcase_bottom.md' %}
