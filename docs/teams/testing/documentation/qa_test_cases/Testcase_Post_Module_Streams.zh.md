---
title: QA:Testcase 模块流 (Module Streams)
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

## 描述

此测试用例负责测试模块流 (module streams)，确保所有模块流均可安装、可用且按预期工作。

## 准备工作

此测试需要单独安装每个模块流，因此最好配置一个新系统并在初始设置后创建快照。之后每次安装模块时都可以回滚到快照。

使用 Minimal Install 组配置系统即可。

## 测试方法

1. 登录机器
2. 获取所有模块流的列表，并与 [RHEL 的流列表](https://access.redhat.com/support/policy/updates/rhel-app-streams-life-cycle) 以及 [Git 源码仓库](https://git.rockylinux.org/rocky/rocky-module-defaults) 中的源进行对比
3. 测试所有流的最简单方法是安装各个流中的软件包组，例如对于 postgresql：

```bash
dnf module install postgresql
dnf module install postgresql:13
dnf module install postgresql:13/client
```

对每个可用的模块流和软件包组重复第 3 步。

这可以通过例如 Ansible 进行自动化，执行所有的 `install -> rollback -> install -> rollback -> ...` 并通过 Ansible 输出结果。

## 预期结果

所有模块流应可用，并且在安装各个流的任何软件包组时不应出现任何错误。（某些安装可能会显示警告，因为它们与其他流不兼容）

{% include 'teams/testing/qa_testcase_bottom.md' %}
