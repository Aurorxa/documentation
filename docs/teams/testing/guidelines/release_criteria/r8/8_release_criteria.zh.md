---
title: Rocky Linux 8 发布标准
author: Trevor Cooper
contributors: Lukas Magauer
tested_with:
tags:
  - testing
revision_date: 2026-06-02
render_macros: true
---

## Rocky Linux 8 最终发布目标

发布（主版本或次版本）的目标是提供一个稳定可靠的企业级 Linux 版本，满足以下需求：

- 最终用户的需求
- 大小企业的需求

## Rocky Linux 8 最终发布要求

为使 Rocky Linux 向公众发布，构建版本 (compose) 必须满足本文档中列出的所有以下标准。这使得决策过程简单明了，尽可能清晰。本文档仅包含"硬性要求"项目。可选/锦上添花的项目不应包含在此列表中。

系统可能在某些配置下无法满足某个要求。在这些情况下，Release Engineering Team 应自行判断该问题是否应被视为发布阻塞项。他们应考虑受该问题影响的用户数量、情况的严重程度、该问题是否容易规避（对于知情的和不知情的用户均适用），以及该问题是否在当前所基于的 Red Hat Enterprise Linux 版本上游中存在。

!!! info "发布阻塞级别 - 服务器"
    ...表示与服务器功能相关的缺陷 (bug) 可被视为发布阻塞项。这适用于任何提供服务 (service) 的软件包，如 httpd 和 nginx。所有架构均适用。

!!! info "发布阻塞级别 - 桌面"
    ...表示与桌面功能 (GNOME) 相关的缺陷可被视为发布阻塞项。这适用于 x86_64 和 aarch64。其他桌面环境（由 EPEL 或 SIG 提供）不被视为阻塞项。

!!! info "发布阻塞级别 - 镜像"
    ...表示与构建的镜像相关的缺陷可能阻塞发布。这适用于所有架构上的 DVD、minimal 和 boot 镜像。

### 初始化要求

#### 发布阻塞镜像必须能启动

发布阻塞级别的安装程序镜像在通过官方支持的方法写入适当大小的光盘介质或 USB 闪存驱动器时必须能够启动（如适用）。测试团队不负责测试光盘介质，但可以测试并反馈。如果发现缺陷，则视为阻塞项。

??? tldr "光盘介质要求"
    发布阻塞级别的镜像在写入适当大小的光盘介质时必须能够启动。当前大小要求为：boot.iso = 789M，minimal.iso = 2.0G，dvd.iso = 10G。

??? tldr "官方支持的 USB 闪存驱动器写入方法"
    - 以下 USB 闪存驱动器写入方法受到官方支持：`dd`
    - 以下 USB 闪存驱动器写入方法_**不**_受支持：`rufus`

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Boot Methods Boot ISO](../../../documentation/qa_test_cases/Testcase_Boot_Methods_Boot_Iso.md)
        - [QA:Testcase Boot Methods DVD](../../../documentation/qa_test_cases/Testcase_Boot_Methods_Dvd.md)
        - [QA:Testcase Media USB dd](../../../documentation/qa_test_cases/Testcase_Media_USB_dd.md)

#### 基本图形模式行为

所有发布阻塞级别安装程序上的通用视频驱动选项（"基本图形模式"）必须按预期工作。这意味着启动安装程序或桌面并尝试使用通用驱动程序。在企业级 Linux 内核支持的所有系统和硬件类别上，此配置下不得存在阻止连接安装程序的缺陷。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Basic Graphics Mode](../../../documentation/qa_test_cases/Testcase_Basic_Graphics_Mode.md)

#### 无破损软件包

严重错误，如未声明的冲突、未解决的依赖关系或依赖其他流中软件包的模块，将被视为自动阻塞项。存在潜在的例外情况（例如，freeradius 无法在较旧的 Perl 流上安装，这是一个已知的上游问题）。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Media Repoclosure](../../../documentation/qa_test_cases/Testcase_Media_Repoclosure.md)
        - [QA:Testcase Media File Conflicts](../../../documentation/qa_test_cases/Testcase_Media_File_Conflicts.md)

#### 仓库必须与上游匹配

仓库及其中的软件包应尽可能与上游匹配。值得注意的例外包括 kmods、kpatch，或被视为"间谍软件"的内容，如 insights。来自上游的软件包不应有对 RHSM 的硬依赖，默认内置有此类依赖的软件包应被打补丁移除。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Repo Compare](../../../documentation/qa_test_cases/Testcase_Repo_Compare.md)
        - [QA:Testcase Packages No Insights](../../../documentation/qa_test_cases/Testcase_Packages_No_Insights.md)
        - [QA:Testcase Packages No RHSM](../../../documentation/qa_test_cases/Testcase_Packages_No_RHSM.md)

#### 去品牌化 (Debranding)

不应包含 Red Hat 特定的资产和功能。如果未打补丁移除，则视为自动阻塞项。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Debranding](../../../documentation/qa_test_cases/Testcase_Debranding.md)

### 安装程序要求

#### 介质一致性验证

这意味着安装程序验证安装介质完整性的机制必须能够成功完成（假设介质已正确写入）。如果不是这种情况，应返回失败消息。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Media USB dd](../../../documentation/qa_test_cases/Testcase_Media_USB_dd.md)
        - [QA:Testcase Boot Methods Boot ISO](../../../documentation/qa_test_cases/Testcase_Boot_Methods_Boot_Iso.md)
        - [QA:Testcase Boot Methods DVD](../../../documentation/qa_test_cases/Testcase_Boot_Methods_Dvd.md)

#### 软件包和安装程序源

安装程序必须能够使用所有支持的本地/远程软件包和安装程序源。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Packages and Installer Sources](../../../documentation/qa_test_cases/Testcase_Packages_Installer_Sources.md)

#### NAS (网络附加存储)

安装程序必须能够检测并安装到受支持的 NAS 设备上（如果内核支持且可能）。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Network Attached Storage](../../../documentation/qa_test_cases/Testcase_Network_Attached_Storage.md)

#### 安装界面

安装程序必须能够使用所有支持的 spoke 完成安装。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Installation Interfaces](../../../documentation/qa_test_cases/Testcase_Installation_Interfaces.md)

#### 最小安装

使用通用网络安装镜像（即 `boot-iso`）且未启用任何更新仓库时，安装程序必须能够安装 'Minimal' 软件包集。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Minimal Installation](../../../documentation/qa_test_cases/Testcase_Minimal_Installation.md)

#### Kickstart 安装

Kickstart 安装应成功完成，无论是通过光盘/USB 介质还是通过网络。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Kickstart Installation](../../../documentation/qa_test_cases/Testcase_Kickstart_Installation.md)

#### 磁盘布局

安装程序必须能够使用安装程序提供或支持的任何文件系统或格式组合创建并安装到任何可行的分区布局。不受 EL 内核支持的文件系统不在此测试范围内（包括 btrfs 和 zfs）。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Disk Layouts](../../../documentation/qa_test_cases/Testcase_Disk_Layouts.md)

#### 固件 RAID

安装程序必须能够检测并安装到固件 RAID 设备上。请注意，特定系统的缺陷不计为阻塞项。某些硬件支持很可能已损坏或不可用。DUD (驱动更新磁盘) 不适用于此标准。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Firmware RAID](../../../documentation/qa_test_cases/Testcase_Firmware_RAID.md)

#### 引导加载程序磁盘选择

安装程序必须允许用户选择引导加载程序 (bootloader) 安装在哪个磁盘上，或者如果用户选择，则不安装引导加载程序。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Bootloader Disk Selection](../../../documentation/qa_test_cases/Testcase_Bootloader_Disk_Selection.md)

#### 存储卷大小调整

安装程序中任何用于调整存储卷大小的机制必须正确尝试所请求的操作。这意味着如果安装程序提供调整存储卷大小的方法，则必须使用正确的调整工具和正确的参数。但是，不要求安装程序禁止调整未格式化卷或未知文件系统类型卷的大小。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Storage Volume Resize](../../../documentation/qa_test_cases/Testcase_Storage_Volume_Resize.md)

#### 更新镜像

安装程序必须能够使用从可移动介质或远程软件包源获取的安装程序更新镜像。这包括 DUD (驱动更新磁盘)。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Update Image](../../../documentation/qa_test_cases/Testcase_Update_Image.md)

#### 安装程序帮助

安装程序中任何包含"帮助"文本的元素在被选中时必须显示相应的帮助文档。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Installer Help](../../../documentation/qa_test_cases/Testcase_Installer_Help.md)

#### 安装程序翻译

安装程序必须正确显示所有可用的完整翻译。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Installer Translations](../../../documentation/qa_test_cases/Testcase_Installer_Translations.md)

### 云镜像要求

#### 云提供商发布的镜像

发布阻塞级别的云磁盘镜像必须发布到适当的云提供商（如 Amazon），并且必须成功启动。这也适用于基于 KVM 的实例，如 x86 和 aarch64 系统。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase TBD](../../../documentation/qa_test_cases/Testcase_Template.md)

### 安装后要求

#### 系统服务

安装后所有系统服务必须正常启动，需要不存在硬件的服务除外。此类服务的示例包括：

- sshd
- firewalld
- auditd
- chronyd

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase System Services](../../../documentation/qa_test_cases/Testcase_Post_System_Services.md)

#### 键盘布局

如果为系统配置了特定的键盘布局，则该布局必须在以下场景中使用：

- 解锁存储卷时（通过 LUKS 加密）
- 在 TTY 控制台登录时
- 通过 GDM 登录时
- 登录 GNOME 桌面系统后（如果用户未设置自己的布局配置）

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Keyboard Layout](../../../documentation/qa_test_cases/Testcase_Post_Keyboard_Layout.md)

#### SELinux 错误（服务器）

/var/log/audit/audit.log 中不得存在 SELinux 拒绝 (denial) 日志

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase SELinux Errors on Server installations](../../../documentation/qa_test_cases/Testcase_Post_SELinux_Errors_Server.md)

#### SELinux 和崩溃通知（仅限桌面）

启动时、安装过程中或首次登录时不得出现 SELinux 拒绝通知或崩溃通知。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase SELinux Errors on Desktop clients](../../../documentation/qa_test_cases/Testcase_Post_SELinux_Errors_Desktop.md)

#### 默认应用程序功能（仅限桌面）

可在 GNOME 内或命令行启动的应用程序必须成功启动，并通过基本功能测试。包括：

- Web 浏览器
- 文件管理器
- 软件包管理器
- 图片/文档查看器
- 文本编辑器 (gedit, vim)
- 归档管理器
- 终端模拟器 (GNOME Terminal)
- 问题报告器
- 帮助查看器
- 系统设置

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Application Functionality](../../../documentation/qa_test_cases/Testcase_Post_Application_Functionality.md)

#### 默认面板功能（仅限桌面）

GNOME 的所有元素在常规使用中应正常运行。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase GNOME UI Functionality](../../../documentation/qa_test_cases/Testcase_Post_GNOME_UI_Functionality.md)

#### 双显示器设置（仅限桌面）

在配有两个显示器的计算机上，图形输出应正确显示在两个显示器上。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Multimonitor Setup](../../../documentation/qa_test_cases/Testcase_Post_Multimonitor_Setup.md)

#### 美术作品与资源资产（服务器和桌面）

必须包含建议的最终美术作品（如壁纸和其他资产）。此软件包中的壁纸应作为 GDM 和 GNOME 的默认壁纸。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Artwork and Assets](../../../documentation/qa_test_cases/Testcase_Post_Artwork_and_Assets.md)

#### 软件包和模块安装

软件包（非模块）应可安装，且不应存在冲突或对 Rocky Linux 外部仓库的依赖。

- 默认模块（如 dnf module list 中列出的）应能安装而无需手动启用。
- 模块流 (module streams) 应能切换，其软件包应能安装且无错误或未解决的依赖项。

??? tldr "参考资料"
    - 测试用例：
        - [QA:Testcase Basic Package installs](../../../documentation/qa_test_cases/Testcase_Post_Package_installs.md)
        - [QA:Testcase Module Streams](../../../documentation/qa_test_cases/Testcase_Post_Module_Streams.md)

#### 身份管理服务器设置

应能够设置 IdM 服务器 (FreeIPA)，使用其功能，并连接客户端。

??? tldr "参考资料"
    - 测试用例
        - [QA:Testcase Identity Management](../../../documentation/qa_test_cases/Testcase_Post_Identity_Management.md)

{% include 'teams/testing/rc_content_bottom.md' %}
