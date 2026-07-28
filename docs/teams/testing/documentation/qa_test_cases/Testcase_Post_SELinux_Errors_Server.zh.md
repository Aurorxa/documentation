---
title: QA:Testcase 服务器安装上的 SELinux 错误
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
    此测试用例关联 [Release_Criteria#selinux-errors-server](../../guidelines/release_criteria/r9/9_release_criteria.md#selinux-errors-server) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

本质上是运行文本模式安装较长时间，在使用过程中检查是否出现任何 SELinux 审计 (audit) 消息。

## 准备工作

获取一个装有文本模式安装基础组的合适系统。

在核心安装之外的其他模式运行此测试也可能有益，但这属于长期测试，略微超出本测试的范围。

## 测试方法

1. 设置新机器或获取已安装机器的访问权限
2. 由于此测试主要关于核心系统的稳定性，大多数情况下只需让系统运行几分钟，如果可能的话运行几小时

## 预期结果

1. 通过 `dnf install setroubleshoot-server` 安装 `sealert`
2. 运行 `sealert -a /var/log/audit/audit.log`
3. 不应显示任何错误或拒绝 (denial) 信息

{% include 'teams/testing/qa_testcase_bottom.md' %}
