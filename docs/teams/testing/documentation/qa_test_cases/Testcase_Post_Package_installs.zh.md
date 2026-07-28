---
title: QA:Testcase 基础软件包安装
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
    此测试用例关联 [Release_Criteria#packages-and-module-installation](../../guidelines/release_criteria/r9/9_release_criteria.md#packages-and-module-installation) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

!!! error "引用的发布标准过于笼统且无法测试"
    此测试用例关联的发布标准 [Release_Criteria#packages-and-module-installation](../../guidelines/release_criteria/r9/9_release_criteria.md#packages-and-module-installation) 过于笼统，**必须**修改到足够具体以便可测试。

## 描述

安装多个软件包应无任何问题。

请同时测试以下用例（这基本上是学习安装软件的乐趣，每次以不同方式执行也很有益）：

- httpd
- httpd 带 php 和 ssl
- nginx
- nginx 带 php 和 ssl
- mysql-server
- mysql-server 带安全配置
- mariadb-server
- postgresql-server
- postgresql-server 带安全配置
- 使用以下工具编译软件包：
    - cmake
    - g++
- ipa-server
- ipa-server 带 dns

## 准备工作

获取一个合适的系统，可在其中无任何问题地安装测试的软件包。

## 测试方法

1. 安装一系列软件包或用例

## 预期结果

安装过程无任何问题。

{% include 'teams/testing/qa_testcase_bottom.md' %}
