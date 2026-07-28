---
title: QA:Testcase 更新镜像
author: Al Bowles
contributors: Trevor Cooper
tested_with: 8.10, 9.7, 10.1
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
    此测试用例关联 [Release_Criteria#Update Image](../../guidelines/release_criteria/r9/9_release_criteria.md#update-image) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述
<!-- TODO 提供有关 updates.img 主题的文档 -->
此测试用例验证**更新镜像 (update image)** 能否加载到 Anaconda 中并在安装过程中应用。

{% include 'teams/testing/qa_data_loss_warning.md' %}

## 准备工作

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

1. 按 Tab 键编辑启动命令

## 测试方法
<!-- TODO 内部托管此文件 -->
1. 在 GRUB 命令行中提供 `inst.updates=https://fedorapeople.org/groups/qa/updates/updates-openqa.img`
1. 照常启动进入安装程序。
1. 在 Anaconda 中，打开"安装目标"页面。

## 预期结果

1. 在"安装目标"页面中，所选安装磁盘应具有粉红色背景
=== "失败 (FAIL)"
    ![未提供更新 - **失败**](../../../../assets/teams/testing/no_updates.png){ loading=lazy }

=== "通过 (PASS)"
    ![已提供更新 - **通过**](../../../../assets/teams/testing/updates.png){ loading=lazy }

1. 如果无法目视验证，请检查 `/tmp/updates` 是否存在，如果更新已成功应用，该目录应包含更新的源文件。请注意，如果更新镜像实际上不包含任何源文件，则不会创建此目录。
<!-- TODO 不完成安装是否也会出现 /tmp/updates？ -->

## 使用 openQA 测试

以下 openQA 测试套件满足此发布标准：

- `install_scsi_updates_img`

## 其他参考资料

- [Red Hat 调试启动选项 (RHEL-9)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/automatically_installing_rhel/custom-boot-options_rhel-installer#debug-boot-options_custom-boot-options)、[Red Hat 调试启动选项 (RHEL-10)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/automatically_installing_rhel/boot-options-reference#debug-boot-options)
- [Fedora QA:Testcase Anaconda updates.img via URL](https://fedoraproject.org/wiki/QA:Testcase_Anaconda_updates.img_via_URL)
- [Fedora QA:Testcase Anaconda updates.img via local media](https://fedoraproject.org/wiki/QA:Testcase_Anaconda_updates.img_via_local_media)

{% include 'teams/testing/qa_testcase_bottom.md' %}
