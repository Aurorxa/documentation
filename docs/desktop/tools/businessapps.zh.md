---
title: 商务与办公应用
author: Ezequiel Bruni
contributors: Steven Spencer, Ganna Zhyrnova
---

## 简介

无论您有一台全新的 Linux 笔记本用于工作，还是正在搭建家庭办公环境，您可能会想知道常用的办公和商务应用在哪里。

它们很多都在 Flathub 上。本指南教您如何安装最常见的这些应用，并提供可行替代方案的清单。继续阅读，了解如何安装 Office、Zoom 等。

## 前提条件

本指南假定以下条件：

* 带有图形桌面环境的 Rocky Linux
* 系统上安装软件的权限
* 已安装并正常运行的 Flatpak 和 Flathub

## 如何在 Rocky Linux 上安装常见的商务软件

对于大多数应用，安装 Flatpak 和 Flathub 后，进入"软件中心"(Software Center)，找到您想要的应用并安装即可。这可以覆盖相当多的常见选项。对于其他的，您需要使用这些应用的浏览器版本。

![软件中心中 Zoom 的截图](images/businessapps-01.png)

为了让您入门，以下是部分最常见商务应用及其桌面客户端和最佳获取方式的清单。

!!! Note

    如果您想了解 Microsoft Office 在 Linux 上的情况，请向下滚动并跳过下一部分。

    此外，此列表不包括 Jira 等没有官方桌面客户端的应用。

### Asana Desktop

桌面应用：Linux 上不可用

推荐：使用 Web 版本。

### Discord

桌面应用：官方和第三方应用均可通过"软件中心"中的 Flathub 获取

推荐：如需按键通话 (push-to-talk)，使用官方客户端。可以使用浏览器版本或"软件中心"中的任何第三方客户端。

### Dropbox

桌面应用：官方应用可通过"软件中心"中的 Flathub 获取。

推荐：在 GNOME 和大多数其他桌面环境中使用官方应用。如果运行 KDE，使用内置的 Dropbox 集成。

### Evernote

桌面应用：不再提供 Linux 版本。

推荐：使用 Web 版本。

### Freshbooks

桌面应用：Linux 上不可用。

推荐：使用 Web 版本。

### Google Drive

桌面应用：第三方客户端。

推荐：在 GNOME Shell 或 KDE 中使用"在线账户"(Online Accounts) 功能登录 Google 账户。在 GNOME 上，这将提供对文件、邮件、日历、待办事项列表等的集成访问。

在 KDE 上，可以通过文件管理器浏览和管理 Drive 文件。集成程度不如 GNOME，但依然有用。

### Hubspot

桌面应用：Linux 上不可用。

推荐：使用 Web 版本。

### Microsoft Exchange

桌面应用：仅有第三方客户端。

推荐：在 GNOME 中，可以使用"在线账户"(Online Accounts) 功能将应用与 Exchange 集成，类似于 Google 账户。

在任何其他桌面环境中，使用 Thunderbird 搭配多个 Exchange 插件之一。Thunderbird 可在默认的 Rocky Linux 仓库中获取，但也可以通过 Flathub 获取更新版本。

### Notion

桌面应用：Linux 上不可用。

推荐：使用 Web 版本。

### Outlook

桌面应用：仅有第三方应用。

推荐：使用您选择的邮件客户端。Evolution 和 Thunderbird 是不错的选择，或者使用 Web 版本。

### Quickbooks

桌面应用：Linux 上不可用。

推荐：使用 Web 版本。

### Slack

桌面应用：可通过"软件中心"中的 Flathub 获取。

推荐：根据偏好使用应用或 Web 版本。

### Teams

桌面应用：可通过"软件中心"中的 Flathub 获取。

推荐：根据需求在桌面或浏览器中使用。如需屏幕共享，启动时请登录 X11 会话。Wayland 上尚未支持屏幕共享。

### Zoom

桌面应用：可通过"软件中心"中的 Flathub 获取。

推荐：如果在 Rocky Linux 上使用桌面应用且需要屏幕共享，请登录 X11 会话而非 Wayland。如今 Wayland 上的屏幕共享已可用，但仅限于较新版本。

Rocky Linux 作为稳定企业操作系统，需要一段时间才能跟进。

不过，根据您的浏览器，在 Wayland 上通过完全跳过桌面应用、直接使用 Web 版本进行屏幕共享可能会获得更好体验。

## 标准商务应用的开源替代方案

如果您可以选择工作和生产力软件，可以考虑改变日常习惯，尝试开源替代方案。上面列出的大多数应用都可以被自托管或云端托管的 [Nextcloud](https://nextcloud.com) 实例以及一些可安装在该平台上的第三方应用所替代。

它可以处理文件同步、项目管理、CRM、日历、笔记管理、基本记账、电子邮件，以及（经过一些工作和配置）文字与视频聊天。

如需更高级、更适合企业使用的 Nextcloud 替代方案，[Wikisuite](https://wikisuite.org/Software) 可以完成上述所有功能，还能帮助您构建公司网站。在这一点上，它与 Odoo 很相似。

但请注意，这些平台主要是以 Web 为中心的。Nextcloud 桌面客户端仅用于文件同步，Wikisuite 则没有桌面客户端。

您可以用 [Mattermost](https://mattermost.com) 轻松替代 Slack，这是一个开源的聊天和团队管理平台。如果需要 Discord、Teams 或 Zoom 提供的音视频功能，可以加入 [Jitsi Meet](https://meet.jit.si)，它有点像可自托管的 Google Meet。

Mattermost 和 Jitsi 在 Flathub 上也都提供 Linux 桌面客户端。

同样，[Joplin](https://joplinapp.org)、[QOwnNotes](https://www.qownnotes.org/) 和 [Notesnook](https://notesnook.com) 都是 Evernote 的出色替代品。

在"软件中心"中寻找 Notion 的替代品？[AppFlowy](https://appflowy.io) 或 [SiYuan](https://b3log.org/siyuan/en/) 可能是您需要的。

!!! Note

    虽然上面列出的每个替代应用都是开源的，但并非所有都是"自由开源软件 (FLOSS)"。这意味着有些会对额外功能或高级版本收费。

## Rocky Linux 上的 Microsoft Office

Linux 世界的新人可能会想，这件事有什么难的。如果您可以使用 Office365 的 Web 版本，那并不难。

然而，如果您需要完整的桌面体验以及 Windows 应用所提供的所有功能，那将更具挑战。虽然偶尔有人写教程说明如何用 WINE 在 Linux 上运行最新版本的 Office 应用，但这些方案通常很快就会失效。目前没有稳定的方式来在 Linux 上运行桌面版 Office 应用。

确实有兼容 Microsoft Office 的 Linux 办公套件，但真正的问题是 Excel。

到目前为止，桌面版 Excel 在功能、数据操作方式等方面几乎无与伦比。不可否认，它是一款其他产品难以复制的庞大软件。

即使替代方案拥有特定用户可能需要的所有功能，工作流也是不同的。您不能将最复杂的公式和电子表格丢进任何一个替代品（甚至包括 Web 版 Excel）中并期望它能正常运作。

但是，如果 Excel 不是您工作流的主要部分，请一定看看这些替代方案。它们*全部*可通过 Flathub 的"软件中心"获取。

### Rocky Linux 上的 Microsoft Office 替代方案

#### LibreOffice

[LibreOffice](https://www.libreoffice.org) 是 FLOSS 办公与生产力软件的事实标准。它涵盖大部分办公需求：文档、电子表格、演示文稿、矢量绘图软件（面向打印场景设计）和数据库。

它与 Microsoft Office 一般具有良好的兼容性，但并不完美，在处理开放格式方面*非常*出色。如果想要完全摆脱 Microsoft 生态系统，LibreOffice 可能是您的最佳选择。

此外，还有一个名为 Collabora Office 的 Web 托管版本，除非付费购买高级版本，否则存在功能限制。

#### OnlyOffice

[OnlyOffice](https://www.onlyoffice.com) 是一个略欠全面但仍然出色的应用套件，用于创建文档、演示文稿、电子表格和 PDF 表单。值得注意的是，它还包含一个 PDF 编辑器。

如果需要 Microsoft Office 兼容性，尤其是文档和演示文稿方面，那么 OnlyOffice 可能是您的最佳选择。OnlyOffice 处理 Word 文档比 Office365 的在线版本更好。

#### WPS Office

[WPS Office](https://www.wps.com)（前身为 Kingsoft Office）已在 Linux 生态系统中存在了相当长的时间。它也支持文档、电子表格、演示文稿和 PDF 编辑器。

WPS Office 与 Microsoft Office 的兼容性略优于 LibreOffice，但不如 OnlyOffice 兼容。它的功能也较少，且可定制性较低。以下是其博客的摘录：

![WPS Office 拥有比 OnlyOffice 更现代、更用户友好的界面，更易于学习和使用，尤其适合初学者。WPS Office 还提供更丰富的模板和主题库，便于创建专业外观的文档。OnlyOffice 比 WPS Office 更强大且更可定制，它拥有更广泛的功能，包括文档管理和项目管理工具。OnlyOffice 对 Microsoft Office 格式的兼容性也优于 WPS Office。](images/businessapps-02.png)

他们的主要重点是创造更简单、更易于访问的用户体验，这也许正是您所需要的。

#### Calligra

[Calligra](https://calligra.org) 办公套件是由 KDE 开发者创建的一个 FLOSS 项目。它提供一套用户友好的基础办公应用，用于创建文档、电子表格、演示文稿、数据库、流程图、矢量绘图、电子书等。

然而，Calligra 应用在 Rocky Linux 上不易安装。如果您有另一台运行 Fedora 的机器，作者建议您可以试试。

## 结语

除了一些显著的例外，在 Rocky Linux 上使用所有办公软件就是在 Flathub 上找到相应应用，或者直接使用 Web 版本。无论如何，对于大多数典型的办公任务，Rocky Linux 很可能是一个稳定且便利的平台。

如果桌面版 Excel 的缺失对您来说是致命的问题，作者建议使用完整的数据库服务器。数据库服务器可以完成一些非常出色的工作。
