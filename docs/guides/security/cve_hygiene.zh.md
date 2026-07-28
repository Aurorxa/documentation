---
title: CVE 卫生管理
author: Howard Van Der Wal
contributors: Steven Spencer
ai_contributors: Claude (claude-opus-4-6)
tested with: 8, 9, 10
tags:
- security
- CVE
- vulnerability
- patching
---

## （或称为什么漏洞扫描器不能被视为安全补丁的最终裁决者）

## AI 使用

本文档遵循[此处提供的 AI 贡献政策。](../contribute/ai-contribution-policy.md)如果您在说明中发现任何错误，请告知我们。

## 简介

漏洞扫描器经常将 Rocky Linux 系统上的软件包标记为未打补丁或过时。在许多情况下，这些是由扫描器解释软件包版本字符串的方式导致的误报。本指南教您如何独立验证某个 CVE（公共漏洞和暴露）是否已在您的 Rocky Linux 软件包中得到修补，理解公告编号，解决扫描器误报，并应用安全更新。

## 前提条件与假设

- 一个 Rocky Linux 8、9 或 10 系统。
- Root 或 `sudo` 访问权限。

## 验证 CVE 补丁的方法

有几种方法可以确定特定 CVE 是否已在您的 Rocky Linux 系统上修复：

- **RPM 变更日志**：查询软件包变更日志中的 CVE 标识符。
- **dnf updateinfo**：检查来自 Rocky Linux 仓库的安全公告。
- **Rocky Linux Errata**：在 [errata.rockylinux.org](https://errata.rockylinux.org/) 搜索公告数据库。
- **上游安全公告**：在 [access.redhat.com/security/security-updates/security-advisories](https://access.redhat.com/security/security-updates/security-advisories) 搜索上游供应商公告数据库。
- **上游 CVE 页面**：在 [access.redhat.com/security/cve/](https://access.redhat.com/security/cve/) 检查平台适用性和严重性。
- **构建系统**：检查 Koji 或 Peridot 以查找待处理的软件包构建。
- **OVAL 扫描**：使用 OpenSCAP 配合 Rocky Linux OVAL 数据进行自动化漏洞评估。

每种方法都有其优势。RPM 变更日志确认本地安装的内容。`dnf updateinfo` 命令显示可用的或已应用的公告。上游安全公告页面和 Rocky Linux Errata 提供了所有已发布修复的最广泛视图。以下各节详细介绍了每种方法。

## 检查 RPM 变更日志中的 CVE 补丁

确定特定 CVE 是否已在已安装的软件包中修复的一种方法是检查其 RPM 变更日志。上游和 Rocky Linux 软件包维护者在回溯安全修复时在变更日志条目中包含 CVE 标识符。

### 基本变更日志查询

要检查特定 CVE 是否已在软件包中修补：

```bash
rpm -q --changelog openssl | grep CVE-2024-6119
```

如果 CVE 已被处理，您将看到类似以下的变更日志条目：

```text
- Fix CVE-2024-6119: Possible denial of service in X.509 name checks
  Resolves: CVE-2024-6119
```

如果该命令没有输出，则当前安装的软件包版本中不存在该 CVE 修复。

### 查询特定软件包版本

要检查尚未安装的软件包文件的变更日志：

```bash
rpm -qp --changelog ./openssl-3.0.7-28.el9_4.x86_64.rpm | grep CVE-2024-6119
```

### 无需安装即可查询仓库

要检查仓库中可用软件包的变更日志：

```bash
dnf repoquery --changelog openssl | grep CVE-2024-6119
```

!!! note "变更日志截断"

    RPM 二进制软件包可能会截断变更日志。构建系统会根据时间修剪旧条目。如果您正在搜索较旧的 CVE，而变更日志搜索没有返回结果，则该条目可能已被修剪。请检查源代码 RPM 或使用 Koji 构建系统 Web 界面，该界面在每个构建页面的底部显示完整的变更日志。

## 使用 dnf updateinfo 获取安全公告

`dnf updateinfo` 子系统提供了影响您系统的安全公告的结构化视图。

### 查看可用公告摘要

```bash
dnf updateinfo
```

示例输出：

```text
Updates Information Summary: available
    3 Security notice(s)
        1 Critical Security notice(s)
        2 Important Security notice(s)
    2 Bugfix notice(s)
    1 Enhancement notice(s)
```

### 列出可用的安全公告

```bash
dnf updateinfo list security
```

### 列出已安装的安全公告

```bash
dnf updateinfo list security --installed
```

### 检查特定 CVE

要查看特定 CVE 是否存在公告：

```bash
dnf updateinfo info --cve CVE-2024-6119
```

要检查修复是否已安装：

```bash
dnf updateinfo list --cve CVE-2024-6119 --installed
```

### 检查可用安全更新而不安装

```bash
dnf check-update --security
```

!!! tip "快速参考"

    | 命令 | 用途 |
    | --------- | --------- |
    | `dnf updateinfo` | 可用公告摘要 |
    | `dnf updateinfo list security` | 列出可用安全公告 |
    | `dnf updateinfo list security --installed` | 列出已安装的安全公告 |
    | `dnf updateinfo info --cve CVE-XXXX-XXXXX` | 特定 CVE 的详细信息 |
    | `dnf check-update --security` | 检查可用的安全更新 |

## 理解 RHSA 和 RLSA 公告编号

Rocky Linux 安全公告（RLSA）直接镜像上游安全公告（RHSA）。公告编号是共享的，只是前缀不同。

例如：

- **RHSA-2024:2551** 是上游公告。
- **RLSA-2024:2551** 是对应的 Rocky Linux 公告。

### 公告类型前缀

| 上游 | Rocky Linux | 含义 |
| --------- | ------------- | --------- |
| RHSA | RLSA | 安全公告 |
| RHBA | RLBA | Bug 修复公告 |
| RHEA | RLEA | 增强公告 |

年份后的序列号在所有公告类型中共享。RHSA 编号可能看起来跳跃，因为中间的编号属于 RHBA 和 RHEA 公告。

### 在哪里查找 Rocky Linux 公告

Rocky Linux 公告发布在 [Rocky Linux Errata](https://errata.rockylinux.org/)。您可以按公告 ID、软件包名称或 CVE 标识符进行搜索。

!!! note "发布时机"

    Rocky Linux 公告可能比相应的上游公告出现得晚。时间延迟取决于 Rocky Linux 发布工程流水线。缺少 RLSA 并不一定意味着修复不可用。始终检查 RPM 变更日志和下文"监控 Rocky Linux 构建系统"部分中描述的构建系统。

## 模块流版本命名

Rocky Linux 8 使用模块化仓库（AppStream），其中软件包发布在发布字符串中包含模块流标识符。此标识符是漏洞扫描器误报的常见来源。

### 模块化发布字符串的结构

典型的模块化软件包发布字符串如下所示：

```text
pcs-0.10.12-6.el8.6.0+7105+f31cb332.x86_64
```

分解如下：

| 组件 | 含义 |
| ----------- | --------- |
| `pcs` | 软件包名称 |
| `0.10.12` | 上游版本 |
| `6` | RPM 发布号 |
| `el8` | Enterprise Linux 8 |
| `.6.0` | 为 8.6 次版本构建的模块流 |
| `+7105` | 模块构建序列号 |
| `+f31cb332` | 标识模块构建的上下文哈希 |

### 扫描器为何错误标记这些

漏洞扫描器将完整的 NVR（Name-Version-Release，名称-版本-发布）字符串与其漏洞数据库进行比较。当扫描器在发布字符串中遇到 `el8.6.0` 时，它可能将其解释为"为 Rocky 8.6 构建"，并将其标记为与发布字符串中包含 `el8.10.0` 的软件包相比已过时。

**这是误报。** 模块流后缀（`el8.6.0` 与 `el8.10.0`）指示模块的构建时间，而不是软件包的安全内容。发布字符串中包含 `el8.6.0` 的软件包可以包含与 `el8.10.0` 完全相同的安全补丁。

### 如何验证

比较实际的软件包版本和 RPM 发布号（模块流后缀之前的部分），而不是完整的 NVR 字符串：

```bash
rpm -q pcs
```

如果两个系统显示：

- `pcs-0.10.12-6.el8.6.0+7105+f31cb332`
- `pcs-0.10.12-6.el8.10.0+1234+abcdef01`

软件包版本（`0.10.12`）和 RPM 发布（`6`）是相同的。这些软件包包含相同的安全补丁，无论模块流后缀如何。

!!! warning "扫描器配置"

    如果您的漏洞扫描器反复将模块流版本差异标记为漏洞，请与您的扫描器供应商合作添加适当的 Rocky Linux 软件包映射。依赖通用 NVD 源而不理解企业 Linux 回溯的扫描器将产生不准确的结果。

## 监控 Rocky Linux 构建系统

当上游已宣布 CVE 修复但在 Rocky Linux 仓库中尚不可用时，您可以跟踪构建和发布流水线。

### 构建系统 URL

| 系统 | URL | 用途 |
| -------- | ----- | --------- |
| Koji | [koji.rockylinux.org](https://koji.rockylinux.org/koji/) | Rocky Linux 8 的构建系统 |
| Peridot | [peridot.build.resf.org](https://peridot.build.resf.org/) | Rocky Linux 9 的构建系统 |
| Errata | [errata.rockylinux.org](https://errata.rockylinux.org/) | 公告数据库 |
| Git | [git.rockylinux.org](https://git.rockylinux.org/) | 源代码软件包仓库 |

### 在 Koji 中检查构建状态

对于 Rocky Linux 8 软件包：

1. 导航到 [koji.rockylinux.org](https://koji.rockylinux.org/koji/)
2. 搜索软件包名称
3. 点击最新构建以查看完整变更日志
4. 检查 CVE 修复是否出现在变更日志中

!!! note "构建完成与仓库可用性"

    软件包出现在 Koji 或 Peridot 构建系统中并不意味着它在官方仓库中可用。软件包在构建后，会经过测试和暂存，然后才发布。使用 `dnf check-update` 检查官方仓库以确认可用性。

## 应用安全更新

### 更新所有安全软件包

```bash
sudo dnf update --security
```

### 应用特定 CVE 的修复

```bash
sudo dnf update --cve CVE-2024-6119
```

### 应用特定公告

```bash
sudo dnf update --advisory RLSA-2024:2551
```

### 最小化安全更新

要应用解决安全问题的最小版本更改：

```bash
sudo dnf upgrade-minimal --security
```

### 配置自动安全更新

安装 `dnf-automatic` 软件包：

```bash
sudo dnf install dnf-automatic
```

编辑 `/etc/dnf/automatic.conf` 以启用仅安全更新：

```ini
[commands]
upgrade_type = security
apply_updates = yes
```

启用并启动定时器：

```bash
sudo systemctl enable --now dnf-automatic.timer
```

!!! warning "生产环境中的自动更新"

    自动安全更新可能会导致意外的服务重启或兼容性问题。在生产环境中，考虑使用 `apply_updates = no` 并在应用前手动审查可用的更新。

## 已终止支持的操作系统

已达到生命周期终止（EOL）的操作系统不再接收安全补丁。如果您的漏洞扫描器在 EOL 系统上标记 CVE，则不会发布上游修复。

### 关键 EOL 日期

| 发行版 | EOL 日期 |
| ------------- | ---------- |
| CentOS 7 | 2024 年 6 月 30 日 |
| Rocky Linux 8 | 2029 年 5 月 31 日 |
| Rocky Linux 9 | 2032 年 5 月 31 日 |

!!! danger "EOL 系统不接收补丁"

    在生产环境中运行 EOL 操作系统意味着接受所有未来的安全漏洞而不进行修复。建议的操作是迁移到受支持的操作系统版本。

## 处理漏洞扫描器误报

漏洞扫描器如 Nessus (Tenable)、Wazuh 和 Nexpose 可能因以下原因在 Rocky Linux 上产生误报：

### 误报的常见原因

1. **模块流版本比较**：扫描器将 `el8.6.0` 标记为与 `el8.10.0` 相比已过时（参见"模块流版本命名"部分）

2. **上游版本比较**：扫描器与上游（未修补）版本号进行比较，不考虑企业 Linux 回溯

3. **缺少 Rocky Linux OVAL 数据**：某些扫描器缺乏适当的 Rocky Linux 漏洞映射，并回退到通用 NVD 源

4. **平台不适用 CVE**：扫描器标记公告中的所有 CVE，而不考虑操作系统适用性

### 扫描器发现的验证工作流

当漏洞扫描器报告一个发现时，请遵循以下工作流：

1. **检查 RPM 变更日志**中的 CVE：

    ```bash
    rpm -q --changelog <package> | grep CVE-XXXX-XXXXX
    ```

2. **检查 dnf updateinfo** 中的公告：

    ```bash
    dnf updateinfo info --cve CVE-XXXX-XXXXX
    ```

3. **检查上游 CVE 页面**的平台适用性：

    ```text
    https://access.redhat.com/security/cve/CVE-XXXX-XXXXX
    ```

4. **如果 CVE 已修补**，记录证据（变更日志条目、公告编号）并将扫描器发现标记为误报。

5. **如果 CVE 未修补**，检查构建系统（Koji 或 Peridot）是否有待处理的修复，或应用可用的安全更新。

!!! tip "记录扫描器例外情况"

    维护已验证的误报记录及其证据。此文档有助于简化未来的漏洞扫描审查并支持审计合规流程。

## 使用 OpenSCAP 进行 OVAL 扫描

Rocky Linux 发布 OVAL（开放漏洞评估语言）数据，可与 OpenSCAP 一起用于执行自动化漏洞评估。

### 下载 Rocky Linux OVAL 数据

OVAL 定义文件可在 [dl.rockylinux.org/pub/oval/](https://dl.rockylinux.org/pub/oval/) 获取：

- Rocky Linux 8：`org.rockylinux.rlsa-8.xml`
- Rocky Linux 9：`org.rockylinux.rlsa-9.xml`

下载与您版本对应的文件：

```bash
wget https://dl.rockylinux.org/pub/oval/org.rockylinux.rlsa-9.xml.bz2
bunzip2 org.rockylinux.rlsa-9.xml.bz2
```

### 运行 OVAL 扫描

如果尚未安装，请安装 OpenSCAP：

```bash
sudo dnf install openscap-scanner
```

运行漏洞扫描：

```bash
oscap oval eval --results oval-results.xml org.rockylinux.rlsa-9.xml
```

生成 HTML 报告：

```bash
oscap oval generate report oval-results.xml > oval-report.html
```

!!! warning "OVAL 数据限制"

    OVAL 扫描可能在 Rocky Linux 上产生误报，特别是对于 OVAL 定义尚未更新以反映最新公告状态的软件包。始终与 RPM 变更日志和 `dnf updateinfo` 交叉引用 OVAL 结果以进行确认。

## 结论

在 Rocky Linux 上进行有效的 CVE 管理需要理解回溯、公告编号和模块流版本控制的工作原理。漏洞扫描器是有价值的工具，但其发现必须通过 RPM 变更日志、`dnf updateinfo` 和官方公告数据库进行验证。通过遵循本指南中的验证工作流，您可以准确地区分真正的漏洞和误报。

## 参考文献

1. "Rocky Linux Errata" by the Rocky Enterprise Software Foundation [https://errata.rockylinux.org/](https://errata.rockylinux.org/)
2. "Security Updates and Advisories" by Upstream [https://access.redhat.com/security/security-updates/security-advisories](https://access.redhat.com/security/security-updates/security-advisories)
3. "CVE Database" by Upstream [https://access.redhat.com/security/cve/](https://access.redhat.com/security/cve/)
4. "Rocky Linux Koji Build System" by the Rocky Enterprise Software Foundation [https://koji.rockylinux.org/koji/](https://koji.rockylinux.org/koji/)
5. "Peridot Build System" by the Rocky Enterprise Software Foundation [https://peridot.build.resf.org/](https://peridot.build.resf.org/)
6. "Rocky Linux Git" by the Rocky Enterprise Software Foundation [https://git.rockylinux.org/](https://git.rockylinux.org/)
7. "Rocky Linux OVAL Data" by the Rocky Enterprise Software Foundation [https://dl.rockylinux.org/pub/oval/](https://dl.rockylinux.org/pub/oval/)
8. "CIQ Security Advisories" by CIQ [https://github.com/ctrliq/advisories/tree/main](https://github.com/ctrliq/advisories/tree/main)
9. "Severity Ratings" by Upstream [https://access.redhat.com/security/updates/classification](https://access.redhat.com/security/updates/classification)
10. "Enterprise Linux Life Cycle - Update Policies" by Upstream [https://access.redhat.com/support/policy/updates/errata](https://access.redhat.com/support/policy/updates/errata)
11. "OpenSCAP Portal" by the OpenSCAP Project [https://www.open-scap.org/](https://www.open-scap.org/)
12. "DNF Automatic" by the DNF Development Team [https://dnf.readthedocs.io/en/latest/automatic.html](https://dnf.readthedocs.io/en/latest/automatic.html)
13. "Explaining Errata" by Upstream [https://access.redhat.com/articles/explaining_redhat_errata](https://access.redhat.com/articles/explaining_redhat_errata)
14. "Rocky Linux Version Information" by the Rocky Enterprise Software Foundation [https://wiki.rockylinux.org/rocky/version/](https://wiki.rockylinux.org/rocky/version/)
