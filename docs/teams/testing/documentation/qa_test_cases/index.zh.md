---
title: QA:测试用例
author: Trevor Cooper
contributors: Bob Robison
tested_with:
tags:
  - testing
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
render_macros: true
---

本页面列出了所有进行中的测试用例及其当前/最后的负责人员。如果某项的 Assignee（负责人）显示为 `@tbd`，则表示我们需要协助编写该测试用例的文档（通常是手动操作），以便未来任何团队成员都能执行。特定的测试用例可以在 [os-autoinst-distri-rocky](https://github.com/rocky-linux/os-autoinst-distri-rocky/issues) 仓库中创建相应 issue 后，在 openQA 中实现。

## 初始化要求

| 要求                                                                                                                                                                                                                                                              | 测试用例                                                                                                                            | 负责人   | 状态                                  |
|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|----------|-----------------------------------------|
| 发布阻塞镜像必须能启动[{{ rc.prod }} 8](../../guidelines/release_criteria/r8/8_release_criteria.md#release-blocking-images-must-boot) [{{ rc.prod }} 9](../../guidelines/release_criteria/r9/9_release_criteria.md#release-blocking-images-must-boot)           | [QA:Testcase Boot Methods Boot ISO](Testcase_Boot_Methods_Boot_Iso.md)                                                             | @tcooper | 模板已存在，openQA 已覆盖 (ref)   |
| 发布阻塞镜像必须能启动[{{ rc.prod }} 8](../../guidelines/release_criteria/r8/8_release_criteria.md#release-blocking-images-must-boot) [{{ rc.prod }} 9](../../guidelines/release_criteria/r9/9_release_criteria.md#release-blocking-images-must-boot)           | [QA:Testcase Boot Methods DVD](Testcase_Boot_Methods_Dvd.md)                                                                       | @tcooper | 模板已存在，openQA 已覆盖 (ref)   |
| 基本图形模式行为[{{ rc.prod }} 8](../../guidelines/release_criteria/r8/8_release_criteria.md#basic-graphics-mode-behaviors)                                                                                                                                      | [QA:Testcase Basic Graphics Mode](Testcase_Basic_Graphics_Mode.md)                                                                 | @tcooper | openQA 测试用例                         |
| VNC 图形模式行为[{{ rc.prod }} 9](../../guidelines/release_criteria/r9/9_release_criteria.md#vnc-graphics-mode-behaviors)                                                                                                                                       | [QA:Testcase VNC Graphics Mode](Testcase_VNC_Graphics_Mode.md)                                                                     | @tcooper | openQA 测试用例                         |
| 无破损软件包[{{ rc.prod }} 8](../../guidelines/release_criteria/r8/8_release_criteria.md#no-broken-packages) [{{ rc.prod }} 9](../../guidelines/release_criteria/r9/9_release_criteria.md#no-broken-packages)                                                    | [QA:Testcase Media Repoclosure](Testcase_Media_Repoclosure.md)[QA:Testcase Media File Conflicts](Testcase_Media_File_Conflicts.md) | @tcooper | 使用脚本手动测试或在 CI 中自动化 |
| 仓库必须与上游匹配[{{ rc.prod }} 8](../../guidelines/release_criteria/r8/8_release_criteria.md#repositories-must-match-upstream) [{{ rc.prod }} 9](../../guidelines/release_criteria/r9/9_release_criteria.md#repositories-must-match-upstream)                  | [QA:Testcase repocompare](Testcase_Repo_Compare.md)                                                                                | @tcooper | 使用 Skip 的 repocompare 手动测试         |
| 去品牌化 (Debranding)[{{ rc.prod }} 8](../../guidelines/release_criteria/r8/8_release_criteria.md#debranding) [{{ rc.prod }} 9](../../guidelines/release_criteria/r9/9_release_criteria.md#debranding)                                                           | [QA:Testcase Debranding Analysis](Testcase_Debranding.md)                                                                          | @tcooper | 使用脚本手动测试或在 CI 中自动化 |

## 安装程序要求

| 要求             | 测试用例                                                                                                                                                                               | 负责人     | 状态                            |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|-----------------------------------|
| 介质一致性验证     | [QA:Testcase Media USB dd](Testcase_Media_USB_dd.md)[QA:Testcase Boot Methods Boot ISO](Testcase_Boot_Methods_Boot_Iso.md)[QA:Testcase Boot Methods DVD](Testcase_Boot_Methods_Dvd.md) | @raktajino |                                   |
| 软件包和安装程序源   | [QA:Testcase Packages and Installer Sources](Testcase_Packages_Installer_Sources.md)                                                                                                   | @raktajino | 已在 openQA 中实现并编写文档 |
| NAS (网络附加存储) | [QA:Testcase Network Attached Storage](Testcase_Network_Attached_Storage.md)                                                                                                           | @tbd       |                                   |
| 安装界面          | [QA:Testcase Installation Interfaces](Testcase_Installation_Interfaces.md)                                                                                                             | @raktajino | 已在 openQA 中实现并编写文档 |
| 最小安装          | [QA:Testcase Minimal Installation](Testcase_Minimal_Installation.md)                                                                                                                   | @raktajino | 已在 openQA 中实现并编写文档 |
| Kickstart 安装   | [QA:Testcase Kickstart Installation](Testcase_Kickstart_Installation.md)                                                                                                               | @raktajino | 已在 openQA 中实现并编写文档 |
| 磁盘布局          | [QA:Testcase Disk Layouts](Testcase_Disk_Layouts.md)                                                                                                                                   | @raktajino | 已在 openQA 中实现并编写文档 |
| 固件 RAID        | [QA:Testcase Firmware RAID](Testcase_Firmware_RAID.md)                                                                                                                                 | @tbd       |                                   |
| 引导加载程序磁盘选择 | [QA:Testcase Bootloader Disk Selection](Testcase_Bootloader_Disk_Selection.md)                                                                                                         | @tbd       |                                   |
| 存储卷大小调整      | [QA:Testcase Storage Volume Resize](Testcase_Storage_Volume_Resize.md)                                                                                                                 | @raktajino | 已在 openQA 中实现并编写文档 |
| 更新镜像          | [QA:Testcase Update Image](Testcase_Update_Image.md)                                                                                                                                   | @raktajino | 已在 openQA 中实现并编写文档 |
| 安装程序帮助       | [QA:Testcase Installer Help](Testcase_Installer_Help.md)                                                                                                                               | @raktajino | 已在 openQA 中实现并编写文档 |
| 安装程序翻译       | [QA:Testcase Installer Translations](Testcase_Installer_Translations.md)                                                                                                               | @tbd       | 已在 openQA 中实现，需编写文档   |

## 云镜像要求

| 要求               | 测试用例                                                                                                             | 负责人   | 状态           |
|-------------------|------------------------------------------------------------------------------------------------------------------------|----------|------------------|
| 云提供商发布的镜像    | [QA:Testcase TBD](Testcase_Template.md)                                                                               | @tbd     |                  |
| Vagrant 镜像正确启动 | [QA:Testcase Vagrant Images - BIOS Boot](Testcase_Vagrant_Images.md#vagrant-file-for-bios-boot)[QA:Testcase Vagrant Images - UEFI Boot](Testcase_Vagrant_Images.md#vagrant-file-for-uefi-boot) | @tcooper | 手动操作，已编写文档 |

## 安装后要求

| 要求                   | 测试                                                                                                                                | 负责人   | 状态                                                               |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------|----------|----------------------------------------------------------------------|
| 系统服务                | [QA:Testcase System Services](Testcase_Post_System_Services.md)                                                                      | @lumarel | 已编写手动指南文档                                              |
| 键盘布局                | [QA:Testcase Keyboard Layout](Testcase_Post_Keyboard_Layout.md)                                                                      | @lumarel | 已在 openQA 中实现并编写文档                                    |
| SELinux 错误（服务器）    | [QA:Testcase SELinux Errors on Server](Testcase_Post_SELinux_Errors_Server.md)                                                       | @lumarel | 已在 openQA 中实现并编写文档                                    |
| SELinux 和崩溃通知（仅限桌面） | [QA:Testcase SELinux Errors on Desktop](Testcase_Post_SELinux_Errors_Desktop.md)                                                  | @lumarel | 部分在 openQA 中实现，已编写文档                             |
| 默认应用程序功能（仅限桌面） | [QA:Testcase Application Functionality](Testcase_Post_Application_Functionality.md)                                                  | @lumarel | 已在 openQA 中实现，另编写手动检查文档 |
| 默认面板功能（仅限桌面）   | [QA:Testcase GNOME UI Functionality](Testcase_Post_GNOME_UI_Functionality.md)                                                        | @lumarel | 已在 openQA 中实现，另编写手动检查文档 |
| 双显示器设置（仅限桌面）   | [QA:Testcase Multimonitor Setup](Testcase_Post_Multimonitor_Setup.md)                                                                | @lumarel | 已编写手动指南文档                                              |
| 美术作品与资源资产（服务器和桌面） | [QA:Testcase Artwork and Assets](Testcase_Post_Artwork_and_Assets.md)                                                          | @lumarel | 已在 openQA 中实现，另编写手动检查文档 |
| 软件包和模块安装        | [QA:Testcase Basic Package installs](Testcase_Post_Package_installs.md)[QA:Testcase Module Streams](Testcase_Post_Module_Streams.md) | @lumarel | 部分在 openQA 中实现，已编写文档                             |
| 身份管理 (FreeIPA)      | [QA:Testcase Identity Management](Testcase_Post_Identity_Management.md)                                                              | @lumarel | 已在 openQA 中实现并编写文档                                    |

{% include 'teams/testing/content_bottom.md' %}
