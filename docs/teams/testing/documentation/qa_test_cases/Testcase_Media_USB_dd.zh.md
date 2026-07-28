---
title: QA:Testcase 介质 USB dd
author: Trevor Cooper
contributors: Lukas Magauer
tested_with: 8.10
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
    此测试用例关联 [Release_Criteria#initialization-requirements](../../guidelines/release_criteria/r9/9_release_criteria.md#initialization-requirements) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

此测试验证 Rocky Linux ISO 镜像可以使用 `dd` 命令写入 USB 介质，并且生成的 USB 介质成功引导进入 Anaconda 安装程序。

!!! error "数据丢失"
    用于此测试的 USB 存储设备上的任何数据很可能被销毁。请不要使用你希望保留内容的 USB 存储设备。

## 设置

1. 提供一个比你要测试的 ISO 镜像更大的 USB 介质设备，并且可以完全擦除。
2. 提供一个具有 `dd` 命令可用且有一个空闲 USB 端口的 Linux（或其他 *nix 系统）。
3. 将你要测试的 Rocky Linux ISO 镜像下载到测试系统上。
    - 示例命令：

   ```bash
   curl -LOR https://dl.rockylinux.org/pub/rocky/10/isos/x86_64/Rocky-10-latest-x86_64-boot.iso
   ```

4. 下载与你要测试的 Rocky Linux ISO 镜像配套的 `CHECKSUM` 文件。
    - 示例命令：

    ```bash
    curl -LOR https://dl.rockylinux.org/pub/rocky/10/isos/x86_64/CHECKSUM
    ```

5. 下载与 `CHECKSUM` 文件配套的 `CHECKSUM.sig` 文件。
    - 示例命令：

    ```bash
    curl -LOR https://dl.rockylinux.org/pub/rocky/10/isos/x86_64/CHECKSUM.asc
    ```

6. 下载 Rocky 发布工程 GPG 密钥。
    - 示例命令：

    ```bash
    curl -LOR https://dl.rockylinux.org/pub/rocky/RPM-GPG-KEY-rockyofficial
    ```

## 如何测试

1. 导入 Rocky 发布工程 GPG 密钥。
    - 示例命令：

    ```bash
    gpg --import RPM-GPG-KEY-rockyofficial
    ```

2. 验证 CHECKSUM 文件的签名。
    - 示例命令：

    ```bash
    gpg --verify-file CHECKSUM.asc
    ```

3. 验证 Rocky Linux ISO 的 CHECKSUM...
    - 示例命令：

    ```bash
    shasum -a 256 --ignore-missing -c CHECKSUM
    ```

4. 使用 `dd` 将 Rocky Linux ISO 写入 USB 介质...
    - 示例命令：

    ```bash
    dd if=Rocky-8.5-x86_64-boot.iso of=/dev/sdX bs=16M status=progress oflag=direct
    ```

    ...其中你需要将 `sdX` 替换为 USB 介质的设备标识符。**这将销毁磁盘上的所有数据。**

5. 使用 USB 介质启动测试系统。
6. 在引导菜单中选择适当的选项以启动安装程序。
7. **[可选]** 在测试系统上继续安装**根据安装程序的选择，这可能会销毁测试系统上的所有数据。**

## 预期结果

1. `CHECKSUM` 文件的 gpg 签名是有效的。
2. Rocky Linux ISO 的 `CHECKSUM` 是有效的。
3. Rocky Linux ISO 被无误地写入 USB 存储设备。
4. USB 存储设备无误地启动。
5. Anaconda 安装程序无误地启动。
6. **[可选]** 安装成功完成，并且如果使用了 minimal 或 DVD ISO，则使用 USB 存储设备上的软件包仓库（而不是基于网络的仓库）进行安装。

!!! error "数据丢失"
    如果你选择完成测试系统的安装，系统上的任何/所有数据可能会丢失。请不要在你需要保留内容的系统上进行安装。

{% include 'teams/testing/qa_testcase_bottom.md' %}
