---
title: QA:Testcase 安装程序翻译
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
    此测试用例关联 [Release_Criteria#Installer Translations](../../guidelines/release_criteria/r9/9_release_criteria.md#installer-translations) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

安装程序必须正确显示所有可用的完整翻译。

## 设置

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

## 如何测试

1. 从 Language Selection（语言选择）spoke 中选择一种语言。

## 预期结果

1. 所有 spoke 应至少显示部分所选语言的内容。
1. 即使选择了不使用拉丁字符的语言，仍可能看到一些以拉丁字符显示的内容，这符合预期。

## 在 openQA 中测试

以下 openQA 测试套件满足此发布标准：

- `install_asian_language`
- `install_arabic_language`
- `install_cyrillic_language`
- `install_european_language`

{% include 'teams/testing/qa_testcase_bottom.md' %}
