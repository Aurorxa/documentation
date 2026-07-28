---
title: QA:Testcase 存储卷大小调整
author: Al Bowles
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

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#Storage Volume Resize](../../guidelines/release_criteria/r9/9_release_criteria.md#storage-volume-resize) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例验证安装程序能否成功调整存储卷上现有分区的大小，或删除并覆盖现有分区。

{% include 'teams/testing/qa_data_loss_warning.md' %}

## 准备工作

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

## 测试方法

### 调整大小

1. 在"安装目标"页面中，在"存储配置"部分，选择"自定义"单选按钮，然后点击"完成"。
1. 点击发行版本和架构左侧的黑色箭头展开可用分区列表。
1. 选择您希望调整大小的分区。如果不希望重新格式化分区，请务必取消选中"重新格式化"复选框。
1. 点击"更新设置"按钮保存设置。
1. 点击 + 按钮在现有分区上创建新分区。提供挂载点和所需容量，然后点击"添加挂载点"。
1. 如有需要可重复以上步骤添加更多分区，或点击"完成"返回 Anaconda 主界面。

### 删除

1. 在"安装目标"页面中，在"存储配置"部分，选择"自动"单选按钮，然后点击"完成"。
1. 您应看到一个"安装选项"对话框，显示可用和可回收的磁盘空间量。
1. 点击"回收空间"按钮。
1. 选择一个分区，然后点击"删除"按钮删除该分区并回收空间。
1. 或者，点击"全部删除"按钮删除所有现有分区。
1. 完成后，点击"回收空间"按钮回收可用的空闲空间。

## 预期结果

1. 安装应完成并成功启动。
1. 调整大小后的分区应正确反映所需的大小。可以使用 `lsblk` 命令进行验证。
1. 已删除的分区应不再存在。

## 在 openQA 中测试

以下 openQA 测试套件满足此发布标准：

- `install_delete_partial`
- `install_delete_pata`
- `install_resize_lvm`

{% include 'teams/testing/qa_testcase_bottom.md' %}
