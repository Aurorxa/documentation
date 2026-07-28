---
title: QA:Testcase 介质仓库对比 (Media Repo Compare)
author: Trevor Cooper
contributors:
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
    此测试用例关联 [Release_Criteria#repositories-must-match-upstream](../../guidelines/release_criteria/r9/9_release_criteria.md#repositories-must-match-upstream) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例将验证仓库及其中的软件包是否尽可能与上游匹配。

## 准备工作

1. 验证是否可以访问 Rocky Linux repocompare 工具。

## 测试方法

1. 访问 [Rocky Linux repocompare 网站](https://repocompare.rockylinux.org/)。
2. 验证 Rocky Linux 仓库与上游内容的相似度。

## 预期结果

1. Rocky Linux 仓库应尽可能与上游仓库匹配。
2. Rocky Linux 软件包的内容应尽可能与上游仓库匹配。

{% include 'teams/testing/qa_testcase_bottom.md' %}
