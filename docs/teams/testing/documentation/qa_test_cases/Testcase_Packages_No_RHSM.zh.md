---
title: QA:Testcase 无 RHSM 软件包
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

此测试将验证来自上游的软件包不应有对 `subscription-manager` (RHSM) 的硬依赖。

## 准备工作

1. 获取可通过 `dnf` 命令操作的环境访问权限。
2. 将要测试的 ISO 下载到该机器。

## 测试方法

1. 在本地挂载要测试的 ISO。
2. 获取对 `subscription-manager` 有 `Requires:` 依赖的软件包列表
    - 示例：

   ```bash
   package_list=($(dnf --refresh repoquery --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath AppStream,/media/AppStream --repo AppStream --whatrequires subscription-manager 2>/dev/null| grep el8))
   ```

3. 下载明确有 `Requires:` 依赖 `subscription-manager` 的软件包
    - 示例：

   ```bash
   dnf --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath AppStream,/media/AppStream --repo AppStream download "${package_list[@]}"
   ```

4. 获取上述软件包的 `SOURCEPKG` 定义
    - 示例：

   ```bash
   rpm -q --queryformat="%{NAME}|%{SOURCERPM}\n" subscription-manager*.rpm | column -s\| -t
   ```

5. 卸载 ISO。

## 预期结果

1. 没有软件包显式依赖 `subscription-manager`。

### 输出示例

=== "成功"

    ```bash
    $ sudo mount -o loop Rocky-8.5-aarch64-minimal.iso /media
    mount: /media: WARNING: device write-protected, mounted read-only.

    $ package_list=($(dnf --refresh repoquery --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath AppStream,/media/AppStream --repo AppStream --whatrequires subscription-manager 2>/dev/null| grep el8))

    $ dnf --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath AppStream,/media/AppStream --repo AppStream download "${package_list[@]}"
    Added BaseOS repo from /media/BaseOS
    Added AppStream repo from /media/AppStream
    Last metadata expiration check: 0:00:25 ago on Sun 24 Apr 2022 10:57:13 PM UTC.

    $ rpm -q --queryformat="%{NAME}|%{SOURCERPM}\n" subscription-manager*.rpm | column -s\| -t
    subscription-manager-cockpit        subscription-manager-1.28.21-3.el8.src.rpm
    subscription-manager-migration      subscription-manager-1.28.21-3.el8.src.rpm
    subscription-manager-plugin-ostree  subscription-manager-1.28.21-3.el8.src.rpm

    $ sudo umount /media
    ```

=== "失败"

    TBD

{% include 'teams/testing/qa_testcase_bottom.md' %}
