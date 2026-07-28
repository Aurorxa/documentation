---
title: QA:Testcase 介质文件冲突
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

此测试用例将验证阻止发布的镜像（release blocking images）上包含的离线仓库中软件包之间是否存在任何未在软件包元数据中通过显式 `Conflicts:` 标签声明的文件冲突。

## 设置

1. 获取对一个环境的访问权限，该环境具有 `dnf` 和 `python3` 命令。
2. 将要测试的 ISO 下载到该机器。
3. 下载 Rocky Linux 测试团队提供的 `potential_conflict.py` 脚本。

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

3. 在挂载的 ISO 上运行 `potential_conflict.py` 脚本。
   - 示例：

   ```bash
   python3 /vagrant/scripts/potential_conflict.py --repofrompath BaseOS,/media/BaseOS --repoid BaseOS --repofrompath Minimal,/media/Minimal --repoid Minimal
   ```

4. 卸载 ISO。
   - 示例：

   ```bash
   umount /media
   ```

## 预期结果

1. `potential_conflict.py` 脚本不报告任何存在未声明冲突的软件包。

### 示例输出

=== "成功"

    ```bash
    $ sudo mount -o loop Rocky-8.5-aarch64-minimal.iso /media
    mount: /media: WARNING: device write-protected, mounted read-only.

    $ python3 /vagrant/scripts/potential_conflict.py \
      --repofrompath BaseOS,/media/BaseOS --repoid BaseOS \
      --repofrompath Minimal,/media/Minimal --repoid Minimal

    Added BaseOS repo from /media/BaseOS
    Added Minimal repo from /media/Minimal
    Getting complete filelist for:
    file:///media/BaseOS
    file:///media/Minimal
    168374 files found.

    Looking for duplicated filenames:
    524 duplicates found.

    Doing more advanced checks to see if these are real conflicts:
     10% complete (    52/   524,  1139/sec),    0 found - eta 0:00:00
     35% complete (   182/   524,  1146/sec),    0 found - eta 0:00:00
     45% complete (   234/   524,  1818/sec),    0 found - eta 0:00:00
     50% complete (   260/   524, 592673/sec),    0 found - eta 0:00:00
     55% complete (   286/   524, 778942/sec),    0 found - eta 0:00:00
     60% complete (   312/   524, 801852/sec),    0 found - eta 0:00:00
     79% complete (   416/   524,   234/sec),    0 found - eta 0:00:00
     84% complete (   442/   524,   902/sec),    0 found - eta 0:00:00
     89% complete (   468/   524,   935/sec),    0 found - eta 0:00:00
     94% complete (   494/   524,  1616/sec),    0 found - eta 0:00:00
     99% complete (   520/   524,  1114/sec),    0 found - eta 0:00:00

    0 file conflicts found.
    0 package conflicts found.

    == Package conflicts ==

    == File conflicts, listed by conflicting packages ==

    $ sudo umount /media
    ```

=== "失败"

    ```bash
    $ sudo mount -o loop Rocky-8.5-x86_64-dvd1.iso /media
    mount: /media: WARNING: device write-protected, mounted read-only.


    $ python3 /vagrant/scripts/potential_conflict.py \
      --repofrompath AppStream,/media/AppStream --repoid AppStream \
      --repofrompath BaseOS,/media/BaseOS --repoid BaseOS

      Added AppStream repo from /media/AppStream
      Added BaseOS repo from /media/BaseOS
      Getting complete filelist for:
      file:///media/AppStream
      file:///media/BaseOS
      851967 files found.

      Looking for duplicated filenames:
      101865 duplicates found.

      Doing more advanced checks to see if these are real conflicts:
        5% complete (  5093/101865,  8713/sec),    0 found - eta 0:00:11
       10% complete ( 10186/101865, 1787281/sec),    0 found - eta 0:00:05
       15% complete ( 15279/101865, 2223312/sec),    0 found - eta 0:00:03
       20% complete ( 20372/101865, 23614/sec),    0 found - eta 0:00:03
       25% complete ( 25465/101865, 57188/sec),    0 found - eta 0:00:02
       30% complete ( 30558/101865,  3831/sec),    0 found - eta 0:00:05
       35% complete ( 35651/101865, 48455/sec),    0 found - eta 0:00:04
       40% complete ( 40744/101865, 32067/sec),    0 found - eta 0:00:03
       45% complete ( 45837/101865, 2136586/sec),    0 found - eta 0:00:03
       50% complete ( 50930/101865, 72529/sec),    0 found - eta 0:00:02
       55% complete ( 56023/101865, 176294/sec),    0 found - eta 0:00:02
       60% complete ( 61116/101865, 68622/sec),    1 found - eta 0:00:01
       65% complete ( 66209/101865, 155133/sec),    1 found - eta 0:00:01
       70% complete ( 71302/101865, 13874/sec),    1 found - eta 0:00:01
       75% complete ( 76395/101865, 10835/sec),    1 found - eta 0:00:01
       80% complete ( 81488/101865, 27477/sec),    1 found - eta 0:00:00
       85% complete ( 86581/101865,  9075/sec),    1 found - eta 0:00:00
       90% complete ( 91674/101865, 14807/sec),    1 found - eta 0:00:00
       95% complete ( 96767/101865, 197437/sec),    1 found - eta 0:00:00
      100% complete (101860/101865, 38727/sec),    1 found - eta 0:00:00

      1 file conflicts found.
      11 package conflicts found.

      == Package conflicts ==
      mariadb-server-utils-3:10.3.28-1.module+el8.4.0+427+adf35707.x86_64
      mysql-server-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64

      python3-mod_wsgi-4.6.4-4.el8.x86_64
      python38-mod_wsgi-4.6.8-3.module+el8.4.0+570+c2eaf144.x86_64
      python39-mod_wsgi-4.7.1-4.module+el8.4.0+574+843c4898.x86_64

      libcmpiCppImpl0-2.0.3-15.el8.i686
      tog-pegasus-libs-2:2.14.1-46.el8.i686

      mariadb-connector-c-devel-3.1.11-2.el8_3.i686
      mariadb-connector-c-devel-3.1.11-2.el8_3.x86_64
      mariadb-devel-3:10.3.28-1.module+el8.4.0+427+adf35707.x86_64
      mysql-devel-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64

      mariadb-server-3:10.3.28-1.module+el8.4.0+427+adf35707.x86_64
      mysql-server-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64

      mariadb-test-3:10.3.28-1.module+el8.4.0+427+adf35707.x86_64
      mysql-test-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64

      mariadb-connector-c-devel-3.1.11-2.el8_3.i686
      mariadb-connector-c-devel-3.1.11-2.el8_3.x86_64
      mysql-devel-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64

      mariadb-devel-3:10.3.28-1.module+el8.4.0+427+adf35707.x86_64
      mysql-devel-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64

      mariadb-3:10.3.28-1.module+el8.4.0+427+adf35707.x86_64
      mysql-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64

      libcmpiCppImpl0-2.0.3-15.el8.x86_64
      tog-pegasus-libs-2:2.14.1-46.el8.x86_64

      libev-libevent-devel-4.24-6.el8.i686
      libev-libevent-devel-4.24-6.el8.x86_64
      libevent-devel-2.1.8-5.el8.i686
      libevent-devel-2.1.8-5.el8.x86_64


      == File conflicts, listed by conflicting packages ==
      mariadb-server-3:10.3.28-1.module+el8.4.0+427+adf35707.x86_64
      mysql-test-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64
        /usr/bin/mysqld_safe

    $ sudo umount /media
    ```

{% include 'teams/testing/qa_testcase_bottom.md' %}
