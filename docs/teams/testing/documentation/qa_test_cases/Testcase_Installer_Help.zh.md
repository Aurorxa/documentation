---
title: QA:Testcase 安装程序帮助
author: Al Bowles
contributors:
tested_with: 8.10, 9.7
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

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#Installer Help](../../guidelines/release_criteria/r9/9_release_criteria.md#installer-help) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

安装程序中任何包含"帮助"文本的元素在被选中时必须显示适当的帮助文档。

## 设置

{% include 'teams/testing/qa_setup_boot_to_media.md' %}

## 如何测试

1. 从 Anaconda Hub，点击右上角的 Help（帮助）按钮。
1. 验证你看到 "Customizing your Installation"（自定义你的安装）帮助页面。
1. 验证 "Configuring language and location settings"（配置语言和位置设置）链接显示主题适当的页面。
1. 关闭 Help 浏览器返回 Anaconda Hub。
1. 验证 Keyboard（键盘）、Language Support（语言支持）和 Time & Date（时间和日期）spoke 显示 Localization（本地化）帮助页面：
    1. 选择 spoke，然后点击 Help（帮助）按钮。
    1. 验证你看到 "Configuring localization options"（配置本地化选项）页面，其中包含一个指向 "Configuring keyboard, language, and time and date settings"（配置键盘、语言以及时间和日期设置）页面的功能链接。
    1. 关闭 Help 浏览器（必要时点击 Done（完成））返回 Anaconda Hub。
1. 验证 Installation Source（安装源）spoke 中的 Help（帮助）按钮显示 "Configuring installation source"（配置安装源）页面。
1. 验证 Software Selection（软件选择）spoke 中的 Help（帮助）按钮显示 "Configuring software selection"（配置软件选择）页面。
1. 验证 Installation Destination（安装目标）spoke 中的 Help（帮助）按钮显示 "Configuring storage devices"（配置存储设备）页面。
1. 验证 Network & Host Name（网络和主机名）spoke 中的 Help（帮助）按钮显示 "Configuring network and host name options"（配置网络和主机名选项）页面。
1. 验证 Root Password（root 密码）spoke 中的 Help（帮助）按钮显示 "Configuring a root password"（配置 root 密码）页面。
1. 验证 User Creation（用户创建）spoke 中的 Help（帮助）按钮显示 "Creating a user account"（创建用户账户）页面。

## 预期结果

1. 所有链接应正常工作并显示相关内容。

## 在 openQA 中测试

以下 openQA 测试套件满足此发布标准：

- `anaconda_help`

{% include 'teams/testing/qa_testcase_bottom.md' %}
