---
title: GNOME 在线账户
author: Ezequiel Bruni
contributors: Steven Spencer
---

## 简介

乍看之下，GNOME 在线账户 (Online Accounts) 功能似乎不起眼，但其实相当强大。它让您能够在几分钟内从桌面应用访问电子邮件、任务、云存储文件、在线日历等。

本简短指南将展示如何入门。

## 前提条件

本指南假定您具备以下条件：

* 已安装 GNOME 桌面环境的 Rocky Linux。

## 如何添加在线账户

打开左上角的 GNOME "活动概览"(Activities Overview)（或按 ++meta++ 或 ++win++ 键），搜索 "Online Accounts"。或者打开"设置"(Settings)面板，在左侧找到"Online Accounts"。

无论哪种方式，您都会看到以下界面：

![GNOME 在线账户设置面板截图](images/onlineaccounts-01.png)

!!! note

    您可能需要点击一个三点垂直图标才能访问此处显示的所有选项：

    ![在线账户面板截图，底部显示三点垂直图标](images/onlineaccounts-02.png)

要添加账户，请点击其中一个选项。对于 Google 账户，您会收到提示，需要在浏览器中登录 Google 并授权 GNOME 访问您的所有数据。对于 Nextcloud 等服务，您将看到如下所示的登录表单：

![Nextcloud 登录表单截图](images/onlineaccounts-03.png)

填写相关信息，GNOME 会负责后续操作。

## GNOME 支持的账户类型

从截图中可以看到，Google、Nextcloud、Microsoft、Microsoft Exchange、Fedora、IMAP/SMTP 和 Kerberos 都得到了一定程度的支持。然而，这些集成的功能并不均等。

Google 账户获得的功能最全面，不过 Microsoft Exchange 和 Nextcloud 也不遑多让。

为了让您清楚地了解哪些功能支持、哪些不支持，以下是作者从 GNOME 官方文档中借用的一张表格：

| **提供商**            | **邮件** | **日历**     | **联系人**     | **地图** | **照片**   | **文件**   | **票据**       |
| -------------------- | -------- | ------------ | ------------ | -------- | ---------- | --------- | ------------- |
| Google               | 是       | 是           | 是           |          | 是         | 是        |               |
| Microsoft            | 是       |              |              |          |            |           |               |
| Microsoft Exchange   | 是       | 是           | 是           |          |            |           |               |
| Nextcloud            |          | 是           | 是           |          |            | 是        |               |
| IMAP 和 SMTP         | 是       |              |              |          |            |           |               |
| Kerberos             |          |              |              |          |            |           | 是            |

!!! Note

    虽然上表中未列出"任务"(tasks)，但它们*似乎*已得到支持，至少对 Google 而言如此。为本指南进行的测试发现，如果您在 Rocky Linux 上安装了 Endeavour 待办事项管理器（可通过 Flathub 获取）且已连接 Google 账户到 GNOME，您的任务将被自动导入。

## 结语

虽然您完全可以使用这些服务的 Web 版本或某些情况下的第三方客户端，但 GNOME 使得将许多最重要的功能直接集成到桌面变得轻松简便。只需登录即可使用。

如果发现有任何服务缺失，请查阅 [GNOME 社区论坛](https://discourse.gnome.org) 并告知他们。
