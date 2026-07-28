---
title: QA:Testcase 应用程序功能
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
    此测试用例关联 [Release_Criteria#default-application-functionality-desktop-only](../../guidelines/release_criteria/r9/9_release_criteria.md#default-application-functionality-desktop-only) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

!!! error "引用的发布标准过于笼统且无法测试"
    此测试用例关联的发布标准 [Release_Criteria#default-application-functionality-desktop-only](../../guidelines/release_criteria/r9/9_release_criteria.md#default-application-functionality-desktop-only) 过于笼统，**必须**修改到足够具体以便可测试。

## 描述

此测试用例涵盖所有被视为 GNOME 桌面环境核心应用程序或面向用户的命令行应用程序。

以下任务通常适用于以下所有应用程序：

- Firefox
- 文件管理器 (Nautilus)
- GNOME 软件中心
- (图片查看器)
- (文档查看器)
- Gedit (文本编辑器)
- 归档管理器
- GNOME 终端 (终端模拟器)
- 问题报告器
- 帮助查看器
- 系统设置
- vim (控制台文本编辑器)

## 准备工作

获取一个装有 Workstation 或 Graphical Server 安装的合适系统。

## 测试方法

1. 检查应用程序是否能无错误地启动
2. 进一步检查右键菜单功能是否正确
3. 打开文件以测试各个应用程序的功能

## 预期结果

确保各个应用程序按预期运行。

{% include 'teams/testing/qa_testcase_bottom.md' %}
