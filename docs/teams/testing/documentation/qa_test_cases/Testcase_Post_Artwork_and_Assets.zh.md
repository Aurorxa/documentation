---
title: QA:Testcase 美术作品与资源资产
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
    此测试用例关联 [Release_Criteria#artwork-and-assets-server-and-desktop](../../guidelines/release_criteria/r9/9_release_criteria.md#artwork-and-assets-server-and-desktop) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

整个操作系统中散布着多个品牌美术作品和资源资产，此测试用例负责检查这些资产是否到位，且不会产生任何 UI 错误。此测试仅适用于使用默认桌面环境 GDM 和 GNOME 的安装。

## 准备工作

1. 获取物理机或虚拟机主机的访问权限，以便安装新机器
2. 为要测试的 ISO 准备适当的介质。
    - 示例：[QA:Testcase Media USB dd](Testcase_Media_USB_dd.md)

## 测试方法

1. 启动镜像时，检查 Anaconda 加载之前的加载屏幕上是否显示正确的徽标
2. 查看 [rocky-logos 仓库](https://github.com/rocky-linux/rocky-logos/tree/r8-fedora/anaconda) 中的 Anaconda 图片，并检查所有资产在 Anaconda 中是否正确应用（在安装过程中通常会立即可见）
3. 使用 Workstation 安装集或 Graphical Server 安装系统
4. 操作系统首次启动时，检查启动登录屏幕出现之前的加载屏幕上是否显示正确的徽标
5. 检查启动登录屏幕的徽标和背景
6. 登录后检查桌面背景以及设置菜单中桌面背景的所有可用选项
7. 锁定屏幕并检查浮层中可见的背景
8. 最后检查登录屏幕的徽标和背景

## 预期结果

整个过程中的测试均能成功完成。

{% include 'teams/testing/qa_testcase_bottom.md' %}
