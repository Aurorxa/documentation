---
title: QA:Testcase Kickstart 安装
author: Al Bowles
contributors: Lukas Magauer
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
    此测试用例关联 [Release_Criteria#Kickstart Installation](../../guidelines/release_criteria/r9/9_release_criteria.md#kickstart-installation) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例验证通过本地和远程 Kickstart 配置文件进行的安装是否成功。

{% include 'teams/testing/qa_data_loss_warning.md' %}

## 设置

1. 将有效的 Kickstart 文件复制到 USB 存储设备
1. 将 USB 存储设备连接到测试系统
{% include 'teams/testing/qa_setup_boot_to_media.md' %}
1. 按 Tab 键编辑引导命令
1. 通过提供 GRUB 引导选项 `inst.ks=file:/path/to/local.ks` 来提供本地 Kickstart 文件，或者通过提供 GRUB 引导选项 `inst.ks=https://git.resf.org/testing/createhdds/raw/branch/rocky/server.ks` 来提供远程 Kickstart 文件。

## 如何测试

1. 继续按正常方式启动安装程序。

## 预期结果

1. 安装应完成并成功启动，自动填充 Kickstart 文件中指定的选项。

## 在 openQA 中测试

以下 openQA 测试套件满足此发布标准：

- `install_kickstart_nfs`
- `server_realmd_join_kickstart`
<!-- TODO 提供一个不需要 PARALLEL_WITH= 的测试套件 -->

{% include 'teams/testing/qa_testcase_bottom.md' %}
