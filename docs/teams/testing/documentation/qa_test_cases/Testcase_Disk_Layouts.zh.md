---
title: QA:Testcase 磁盘布局
author: Al Bowles
contributors:
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
    此测试用例关联 [Release_Criteria#Disk Layouts](../../guidelines/release_criteria/r9/9_release_criteria.md#disk-layouts) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例验证在任何受支持的分区布局上使用任何文件系统或格式组合的成功安装。

{% include 'teams/testing/qa_data_loss_warning.md' %}

## 设置

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

## 如何测试

1. 选择 Installation Destination（安装目标）spoke。
1. 选择应安装操作系统的卷。
1. 在 Storage Configuration（存储配置）部分下选择 Custom（自定义）单选按钮，然后点击 "Done"（完成）。
1. 对每个卷，执行以下步骤：
    1. 从下拉菜单中选择所需的分区方案。支持的选项有 Standard Partition（标准分区）、LVM 和 LVM Thin Provisioning（LVM 精简配置）。
    1. 选择 "Encrypt my data"（加密我的数据）复选框以创建加密文件系统。
    1. 选择左下角的加号 (+) 按钮以添加分区。
    1. 定义所需的挂载点和卷容量，然后点击 "Add mount point"（添加挂载点）。
    1. 设置设备类型。支持的选项有 LVM、RAID、Standard Partition（标准分区）和 LVM Thin Provisioning（LVM 精简配置）。
    1. 如果设备类型设置为 RAID，选择 RAID 级别。支持的选项有 RAID0、RAID1、RAID4、RAID5、RAID6 和 RAID10。
    1. 设置文件系统类型。支持的选项有 BIOS Boot、ext2、ext3、ext4、swap、vfat 和 xfs。
    1. 在受支持的情况下，你可以通过取消 Reformat（重新格式化）复选框来选择不对现有分区进行格式化。
1. 当所有分区创建完成后，点击左上角的蓝色 Done（完成）按钮。
1. 查看 Summary of Changes（变更摘要）对话框，然后点击 Accept Changes（接受变更）。
1. 按正常方式继续安装。

## 预期结果

1. 安装应成功完成并引导到适当的磁盘。
1. 应使用指定的文件系统类型和分区方案。
1. 如果已配置，软件 RAID 应按预期运行。

## 使用 openQA 进行测试

以下 openQA 测试套件满足此发布标准：

- `install_standard_partition_ext4`
- `install_custom_gui_standard_partition_ext4`
- `install_lvm_ext4`
- `install_custom_gui_lvm_ext4`
- `install_software_raid`
- `install_custom_gui_software_raid`
- `install_xfs`
- `install_custom_gui_xfs`
- `install_lvmthin`
- `install_multi`
- `install_multi_empty`

{% include 'teams/testing/qa_testcase_bottom.md' %}
