---
title: QA:Testcase 软件包和安装源
author: Al Bowles
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
    此测试用例关联 [Release_Criteria#Packages and Installer Sources](../../guidelines/release_criteria/r9/9_release_criteria.md#packages-and-installer-sources) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例验证安装程序能够通过任何受支持的安装源成功安装任何受支持的软件包集。

以下软件包集支持从本地介质安装：

- server
- minimal

以下软件包集仅从远程源获得，需要网络连接：

- workstation
- graphical-server
- virtualization-host

{% include 'teams/testing/qa_data_loss_warning.md' %}

## 设置

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

## 如何测试

1. 对于本地软件包安装，不需要启用网络或指定镜像。
1. 对于从远程源安装软件包：
    1. 从 Network and Hostname（网络和主机名）spoke 中，启用网络。
    1. 从 Installation Source（安装源）spoke 中，配置远程软件源，提供适合被测版本和架构的适当[镜像](https://mirrors.rockylinux.org)。
1. 完成安装程序并等待机器重新启动。

## 预期结果

1. 安装应完成并成功启动。
1. 如果指定了图形化软件包集，系统应引导到图形登录界面。

## 在 openQA 中测试

以下 openQA 测试套件满足此发布标准，前提是它们至少通过 `_do_install_reboot` 模块：

- `install_packageset_server`
- `install_packageset_minimal`
- `install_packageset_workstation`
- `install_packageset_graphical-server`
- `install_packageset_virtualization-host`

{% include 'teams/testing/qa_testcase_bottom.md' %}
