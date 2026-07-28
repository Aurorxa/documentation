---
title: 在 GNOME 中使用 Valuta 进行货币换算
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - gnome
  - desktop
  - currency
  - flatpak
---

## 简介

在这个数字交易跨越各大洲、旅行只需一张机票的日益互联的世界中，快速可靠的货币换算需求至关重要。无论您是经验丰富的环球旅行者、精明的在线购物者，还是管理国际财务的远程专业人士，专业的货币换算器都是不可或缺的工具。对于 GNOME 桌面用户而言，Valuta 展现出了优雅简洁与专注功能的典范。

Valuta 不单是一个计算器，它是一款经过精心设计、与 GNOME 生态系统无缝集成的应用程序。它摒弃了不必要的复杂性，提供直接高效的用户体验，干净利落地完成工作。极简的界面以及对 GNOME 设计原则的遵循，使其感觉像是桌面的原生扩展，提供对最新汇率的即时访问。

## 安装：将 Valuta 纳入您的 GNOME 桌面

Valuta 以 Flatpak 格式分发，这种现代打包格式可确保应用程序在各 Linux 发行版中保持一致运行，提供沙箱保护以增强安全性，并允许直接从开发者处获取最新版本。

要安装 Valuta，打开终端并运行以下命令：

```bash
flatpak install flathub io.github.ideveCore.Valuta
```

该命令将从 Flathub（通用 Flatpak 应用商店）获取并安装 Valuta。安装完成后，可以直接从应用程序菜单启动 Valuta，或在终端中输入 `flatpak run io.github.ideveCore.Valuta`。

## 使用 Valuta：简洁实用

Valuta 的优势在于其直观的设计。应用程序聚焦于单一核心任务：以最少的交互将一种货币换算为另一种。

1. **选择基准货币：** 启动 Valuta 后，通常会看到两个货币字段。顶部字段代表"基准"货币。点击货币代码（如 "USD"）打开搜索栏，可以输入并从中选择所需货币。
2. **输入金额：** 在基准货币旁的输入框中，输入要换算的金额。随着您的输入，Valuta 将在第二个货币字段中即时显示换算后的值。
3. **选择目标货币：** 类似地，点击底部字段中的货币代码，选择要换算到的目标货币。Valuta 将立即根据最新汇率更新换算后的金额。

该应用程序自动从欧洲中央银行（通过 Frankfurter）和 Moeda.info 等可靠来源获取并更新汇率，确保您随时掌握准确信息。为了更快速的换算，Valuta 集成了 GNOME 搜索提供程序，允许直接从 GNOME "活动概览"(Activities Overview) 进行换算（例如输入 "10 USD to EUR"）。

## 超越 GNOME：其他平台上的替代方案

对于从其他平台迁移到 Rocky Linux 桌面的用户，Valuta 是一个称职的替代品。以下是 macOS 和 Windows 11 平台上一些值得注意的货币换算应用程序，Valuta 可作为其合适的替代：

### 针对 macOS

* **Converter (Currency Converter Calculator):** 该应用程序提供熟悉的计算器式界面，用于快速高效的货币换算，以易用性和与 macOS 美学风格的集成而备受赞誉。
* **XE Currency Converter:** XE 作为全球公认的货币兑换品牌，提供功能强大的应用程序，具有实时汇率、离线功能和用户友好的界面。它是适合日常和严肃货币追踪的综合工具。

### 针对 Windows 11

* **Windows Calculator App:** Windows 11 内置的"计算器"(Calculator)应用程序包含一个功能出奇强大的货币换算器。它支持超过 100 种货币，甚至可以离线换算，是便利的默认选项。
* **Callista Currency Converter:** 可在 Microsoft Store 上获取，Callista 提供最新的实时汇率，支持大量货币，通常还包含换算历史等功能，且免费无广告。
* **XE Currency Converter:** 与 macOS 一样，XE 也为 Windows 用户提供了强大的货币换算工具，具备实时汇率、汇款选项和汇率提醒功能。

## 结语

Valuta 证明了 GNOME 生态系统中专注设计的力量。它提供快速、准确且美观的货币换算解决方案，无缝融入您的桌面工作流。对于追求工具效率与优雅的人而言，Valuta 是一个极佳的选择，确保管理国际财务或规划下一次冒险只需几下点击。
