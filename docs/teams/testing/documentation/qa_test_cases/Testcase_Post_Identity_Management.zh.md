---
title: QA:Testcase 身份管理
author: Lukas Magauer
contributors:
tested_with: 8.10, 9.7, 10.1
tags:
  - testing
  - qa
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
  ver:
    - 8
    - 9
    - 10
  level: Final
render_macros: true
---

!!! info "发布适用范围"
    此测试用例适用于以下版本的 {{ rc.prod }}：{% for version in rc.ver %}{{ version }}{% if not loop.last %}, {% endif %}{% endfor %}

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria#identity-management-server-setup](../../guidelines/release_criteria/r10/10_release_criteria.md#identity-management-server-setup) 发布标准。如果您正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

搭建 IdM 系统 (FreeIPA) 并使用其功能不仅利用了官方仓库中的大量软件包，还能测试企业环境中的许多常用功能。此安装将托管自己的 DNS 服务器，以便进行更通用的测试，而无需依赖环境的个别基础设施。

## 环境要求

- 一台新部署的系统（此系统上不允许运行除 IdM 服务以外的其他功能）
- 具有不受管理的域名（安装程序将检查 DNS 服务器）和不受管理的反向 DNS 网络的 IPv4 网络（本例中为 10.30.30.0/24 和 ipa1.network）
- 在本文档的场景中，外部 DNS 服务器的域名为 `example.com`，其中必须包含 `r10-ipa1-dev.example.com` 条目（如果不涉及外部 DNS 服务器，也可以用 `/etc/hosts` 文件中的条目替代）

## 准备工作

1. Rocky Linux 版本：

    - 8：`dnf module install idm:DL1/dns`
    - 9/10：`dnf install ipa-server-dns`

2. `ipa-server-install`

    - Do you want to configure integrated DNS (BIND)? [no]: yes
    - Server host name [r10-ipa1-dev.example.com]: `留空`
    - Please confirm the domain name [example.com]: ipa1.network
    - Please provide a realm name [IPA1.NETWORK]: `留空`
    - Directory Manager password: `<密码>`
      Password (confirm): `<密码>`
    - IPA admin password: `<其他密码>`
      Password (confirm): `<其他密码>`
    - （仅 8）Please provide the IP address to be used for this host name: 10.30.30.1
    - （仅 8）Enter an additional IP address, or press Enter to skip: `留空`
    - Do you want to configure DNS forwarders? [yes]: `留空`
    - Do you want to configure these servers as DNS forwarders? [yes]: `留空`
    - Enter an IP address for a DNS forwarder, or press Enter to skip: `留空`
    - Do you want to search for missing reverse zones? [yes]: `留空`
    - NetBIOS domain name [IPA1]: `留空`
    - Do you want to configure chrony with NTP server or pool address? [no]: yes
    - Enter NTP source server addresses separated by comma, or press Enter to skip: `留空`
    - Enter a NTP source pool address, or press Enter to skip: pool.ntp.org
    - Continue to configure the system with these values? [no]: yes

3. `firewall-cmd --add-service={freeipa-4,dns} --permanent`
4. `firewall-cmd --reload`

## 测试方法

1. 通过运行 `kinit admin` 和 `klist` 确认 Kerberos 正常工作
2. 确认 Web 前端可访问并且登录正常
3. 此外，你还可以附加另一台系统（DNS + 通过 SSSD 连接）

## 预期结果

安装后所有服务应可用且正常工作。

{% include 'teams/testing/qa_testcase_bottom.md' %}
