---
title: QA:Testcase 系统服务
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
    此测试用例关联 [Release_Criteria#system-services](../../guidelines/release_criteria/r9/9_release_criteria.md#system-services) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试覆盖检查所有随基础组安装的基础系统服务是否正常启动/运行。

## 准备工作

1. 获取物理机或虚拟机主机的访问权限，以便安装新机器
2. 为要测试的 ISO 准备适当的介质。
    - 示例：[QA:Testcase Media USB dd](Testcase_Media_USB_dd.md)

## 测试方法

启动系统并检查所有服务是否无故障运行：

```bash
systemctl status
```

## 预期结果

整个过程中的测试均能成功完成。

{% include 'teams/testing/qa_testcase_bottom.md' %}
