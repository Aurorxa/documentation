---
title: 'SOP: openQA - System Upgrades (系统升级)'
author: Al Bowles
contributors: Trevor Cooper
tested_with: 8.10, 9.6, 10.0
tags:
  - openQA
  - sop
  - testing
revision_date: 2026-05-08

render_macros: true
---

本 SOP 详细说明在 openQA 主机上执行系统升级的必要步骤。

{% include "teams/testing/contacts_top.md" %}

## Fedora

1. 确认当前系统已完全升级

    ``` bash linenums="1"
    dnf upgrade --refresh
    ```

1. 安装系统升级插件

    ``` bash linenums="1"
    dnf install dnf-plugin-system-upgrade
    ```

1. 下载下一版本的升级软件包

    ``` bash linenums="1"
    dnf system-upgrade download --releasever=[newversion]
    ```

1. 重启进入离线升级模式

    ``` bash linenums="1"
    dnf system-upgrade reboot
    ```

1. 重启后清理

    ``` bash linenums="1"
    dnf system-upgrade clean
    dnf clean packages
    ```

## 升级后任务 (Post-Upgrade Tasks)

以下步骤在某些（但不是所有）情况下也可能是必要的。

### 升级 PostgreSQL 数据库

1. 安装 postgresql-upgrade 软件包

    ``` bash linenums="1"
    dnf install postgresql-upgrade
    ```

1. 升级你的 postgres 数据库

    ``` bash linenums="1"
    sudo -iu postgres
    postgresql-setup --upgrade
    ```

### 重新应用 Rocky 品牌标识

1. 获取 [Ansible openQA 部署仓库](https://git.resf.org/infrastructure/ansible-openqa-management){target=_blank}

1. 运行品牌标识相关任务

    ``` bash linenums="1"
    ansible-playbook init-openqa-rocky-developer-host.yml -t branding
    ```

## 参考资料 (References)

- [使用 DNF 系统升级方式升级 Fedora](https://docs.fedoraproject.org/en-US/quick-docs/dnf-system-upgrade/){target=_blank}
- [如何轻松升级到 Fedora Workstation 36](https://www.makeuseof.com/how-to-upgrade-to-fedora-workstation-36/){target=_blank}

{% include "teams/testing/content_bottom.md" %}
