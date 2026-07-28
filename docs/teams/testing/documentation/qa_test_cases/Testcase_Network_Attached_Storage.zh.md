---
title: QA:Testcase 网络附加存储
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
    此测试用例关联 [Release_Criteria#Network Attached Storage](../../guidelines/release_criteria/r9/9_release_criteria.md#nas-network-attached-storage) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

安装程序必须能够检测受支持的 NAS（网络附加存储）设备并安装到其上（如果可能且内核支持）。

## 设置

1. 添加此测试用例的设置步骤。

## 如何测试

### NFS

install nfs-utils
sudo mount -t nfs nfs-server:/nfs/path /mnt
then a created a file echo 1 > /mnt/1
verified it and permissions ls /mnt; cat /mnt/1
then deleted it rm /mnt/1
then unmounted sudo umount /mnt

### iSCSI

## 预期结果

1. 这是你应该看到/验证的内容。
2. 你还应该看到/验证此内容。

{% include 'teams/testing/qa_testcase_bottom.md' %}
