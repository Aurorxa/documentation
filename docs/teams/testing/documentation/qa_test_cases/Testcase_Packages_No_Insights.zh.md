---
title: QA:Testcase 无 Insights 软件包
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

此测试将验证 `insights-client` 软件包不会被声明为软件包组 (package group) 的一部分进行安装。

## 准备工作

1. 获取可通过 `dnf` 命令操作的环境访问权限。
2. 将要测试的 ISO 下载到该机器。

## 测试方法

1. 在本地挂载要测试的 ISO。
2. 确定 ISO 上 `comps` 文件的路径。
3. 验证 `insights-client` 没有被声明为自动安装。
   - 示例 1：

   ```bash
   find /media -name "*comps*.xml" -exec grep -H "insights-client" '{}' \;
   ```

   - 示例 2：

   ```bash
   dnf --refresh --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath AppStream,/media/AppStream --repo AppStream groupinfo base | grep -E ":|insights"
   ```

4. 卸载 ISO。

## 预期结果

1. `insights-client` 没有被声明为默认安装。

### 输出示例

=== "成功"

    !!! info "更新示例"
        注意：此示例需要在 8.6 ISO 生成后进行刷新。如下面的失败部分所示，`Rocky-8.5-x86_64-dvd1.iso` 将 `insights-client` 作为 `base` 组的一部分。该软件包应包含在 DVD ISO 中，但**不**应自动安装。

    ```bash
    $ sudo mount -o loop Rocky-8.5-aarch64-minimal.iso /media
    mount: /media: WARNING: device write-protected, mounted read-only.

    $ dnf --refresh --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath Minimal,/media/Minimal --repo Minimal search insights-client
    Added BaseOS repo from /media/BaseOS
    Added Minimal repo from /media/Minimal
    BaseOS                                                                    3.8 MB/s | 3.9 kB     00:00
    Minimal                                                                   3.7 MB/s | 3.8 kB     00:00
    No matches found.

    $ find /media -name "*comps*.xml" -exec grep -H "insights-client" '{}' \;

    $ dnf --refresh --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath Minimal,/media/Minimal --repo Minimal groupinfo base | grep -E ":|insights"
    BaseOS                                          3.8 MB/s | 3.9 kB     00:00
    Minimal                                         3.7 MB/s | 3.8 kB     00:00
    Group: Base
     Description: The standard installation of Rocky Linux.
     Mandatory Packages:
     Default Packages:
     Optional Packages:

    $ sudo umount /media
    ```

=== "失败"

    ```bash
    $ sudo mount -o loop Rocky-8.5-x86_64-dvd1.iso /media
    mount: /media: WARNING: device write-protected, mounted read-only.

    $ dnf --refresh --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath AppStream,/media/AppStream --repo AppStream search insights-client
    Added BaseOS repo from /media/BaseOS
    Added AppStream repo from /media/AppStream
    BaseOS                                                                    3.8 MB/s | 3.9 kB     00:00
    AppStream                                                                 4.2 MB/s | 4.3 kB     00:00
    ================================= Name Exactly Matched: insights-client ==================================
    insights-client.noarch : Uploads Insights information to Red Hat on a periodic basis

    $ find /media -name "*comps*.xml" -exec grep -H "insights-client" '{}' \;
    /media/AppStream/repodata/a6742e1300e1c786af91656b152d3b98bb7aff598e650509381417970e1f1b7e-comps-AppStream.x86_64.xml:      <packagereq type="default">insights-client</packagereq>
    /media/AppStream/repodata/a6742e1300e1c786af91656b152d3b98bb7aff598e650509381417970e1f1b7e-comps-AppStream.x86_64.xml:      <packagereq type="default">insights-client</packagereq>

    $ dnf --refresh --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath AppStream,/media/AppStream --repo AppStream groupinfo base | grep -E ":|insights"
    BaseOS                                          3.8 MB/s | 3.9 kB     00:00
    AppStream                                       4.2 MB/s | 4.3 kB     00:00
    Group: Base
     Description: The standard installation of Rocky Linux.
     Mandatory Packages:
     Default Packages:
       insights-client
     Optional Packages:

    $ sudo umount /media
    ```

{% include 'teams/testing/qa_testcase_bottom.md' %}
