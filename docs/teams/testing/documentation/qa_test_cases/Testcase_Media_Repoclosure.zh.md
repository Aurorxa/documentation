---
title: QA:Testcase 介质仓库闭包
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
    此测试用例关联 [Release_Criteria#no-broken-packages](../../guidelines/release_criteria/r9/9_release_criteria.md#no-broken-packages) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试用例将验证阻止发布的镜像上包含的离线仓库不包含损坏的依赖关系。

## 设置

1. 获取对一个环境的访问权限，该环境具有 `dnf repoclosure` 命令。
2. 将要测试的 ISO 下载到该机器。

## 如何测试

1. 将要测试的 ISO 挂载到本地。
    - 示例：

   ```bash
   mount -o loop Rocky-8.5-x86_64-minimal.iso /media
   ```

2. 确定 ISO 上 `repodata` 目录的路径。
    - 示例：

   ```bash
   find /media -name repodata
   ```

3. 在挂载的 ISO 上运行 `dnf repoclosure` 命令。
    - 示例：

   ```bash
   dnf --verbose repoclosure --repofrompath BaseOS,/media/BaseOS --repo BaseOS --repofrompath Minimal,/media/Minimal --repo Minimal
   ```

4. 卸载 ISO。
    - 示例：

   ```bash
   umount /media
   ```

## 预期结果

1. `dnf repoclosure` 命令不会生成任何错误。

### 示例输出

=== "成功"

    ```bash
    $ sudo mount -o loop Rocky-8.5-x86_64-minimal.iso /media
    mount: /media: WARNING: device write-protected, mounted read-only.

    [vagrant@localhost ~]$ dnf --refresh repoclosure \
      --repofrompath BaseOS,/media/BaseOS --repo BaseOS \
      --repofrompath Minimal,/media/Minimal --repo Minimal
    Added BaseOS repo from /media/BaseOS
    Added Minimal repo from /media/Minimal
    BaseOS                                               102 MB/s | 2.6 MB     00:00
    Minimal                                               90 kB/s | 384  B     00:00

    $ sudo umount /media
    ```

=== "失败"

    __注意：在此示例中，`Rocky-8.5-x86_64-minimal.iso` 的内容被复制到 `/tmp`，然后 BaseOS 仓库被修改以移除 `setup-2.12.2-6.el8.noarch.rpm` 软件包，并重新生成了仓库元数据。__

    ```bash
    [vagrant@localhost ~]$ dnf --refresh repoclosure \
      --repofrompath BaseOS,/tmp/media/BaseOS --repo BaseOS \
      --repofrompath Minimal,/tmp/media/Minimal --repo Minimal
    Added BaseOS repo from /tmp/media/BaseOS
    Added Minimal repo from /tmp/media/Minimal
    BaseOS                                               3.7 MB/s | 3.8 kB     00:00
    Minimal                                              3.7 MB/s | 3.8 kB     00:00
    package: basesystem-11-5.el8.noarch from BaseOS
      unresolved deps:
        setup
    package: dump-1:0.4-0.36.b46.el8.x86_64 from BaseOS
      unresolved deps:
        setup
    package: filesystem-3.8-6.el8.x86_64 from BaseOS
      unresolved deps:
        setup
    package: initscripts-10.00.15-1.el8.x86_64 from BaseOS
      unresolved deps:
        setup
    package: rpcbind-1.2.5-8.el8.x86_64 from BaseOS
      unresolved deps:
        setup
    package: shadow-utils-2:4.6-14.el8.x86_64 from BaseOS
      unresolved deps:
        setup
    Error: Repoclosure ended with unresolved dependencies.
    ```

{% include 'teams/testing/qa_testcase_bottom.md' %}
