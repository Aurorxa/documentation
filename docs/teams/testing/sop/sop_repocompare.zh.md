---
title: 'SOP: Repocompare (仓库对比)'
author: Al Bowles
contributors: Trevor Cooper
tested_with: 8.10, 9.6, 10.0
tags:
  - sop
  - testing
revision_date: 2026-05-08
render_macros: true
---

本 SOP 说明如何执行 repocompare (仓库对比) 流程，以确保 Rocky 的软件包仓库与 RHEL 软件包仓库保持同步。

{% include "teams/testing/contacts_top.md" %}

要识别哪些软件包可能需要更新，请访问相应的 [RepoCompare](https://repocompare.rockylinux.org){target=_blank} 页面，重点关注每个版本的 **SRPM Repo Comparison** (SRPM 仓库对比) 页面。
其中 **Rocky** 版本**低于** **RHEL** 版本的软件包很可能需要更新——你可以进行手动对比以确认。

## 环境配置 (Setup)

从一台**具有有效授权 (entitlement) 的 RHEL8 机器**上，获取 repocompare 仓库：

``` bash linenums="1"
git clone https://git.resf.org/testing/repocompare
cd repocompare/
```

导入 Rocky 和 RHEL 的 RPM GPG 密钥

``` bash linenums="1"
curl -O http://dl.rockylinux.org/pub/rocky/RPM-GPG-KEY-Rocky-8
curl -O http://dl.rockylinux.org/pub/rocky/RPM-GPG-KEY-Rocky-9
rpm --import RPM-GPG-KEY-Rocky-8
rpm --import RPM-GPG-KEY-Rocky-9
rpm --import /etc/pki/rpm-gpg/redhat-official
```

## 对比软件包 (Comparing a package)

如果 RHEL 软件包的 Name/Epoch/Version/Release (NEVR) 比 Rocky 软件包更新，则该软件包需要更新。在这种情况下，RHEL 软件包的 changelog (变更日志) 中通常也会有一个更新的条目，如下所示：

``` bash linenums="1"
./manual_compare.sh 9 AppStream golang
Rocky Linux 9.2    golang 1.19.9 2.el9_2 * Tue May 23 2023 Alejandro Sáez <asm@redhat.com> - 1.19.9-2
Red Hat            golang 1.19.10 1.el9_2 * Tue Jun 06 2023 David Benoit <dbenoit@redhat.com> - 1.19.10-1
```

请注意，Red Hat 的 golang 软件包版本号高于 Rocky Linux 9.2 的软件包。其 changelog 中也有一个更新的条目。

## 注意事项 (Gotchas)

某些软件包被认为与 repocompare 无关。这些包括：

``` bash linenums="1"
rhc
shim-unsigned
# 存在于 RHEL 中但不在 Rocky 中的任何软件包（在 repocompare 网站上 Rocky 列中标记为 **DOES NOT EXIST**）
```

{% include "teams/testing/content_bottom.md" %}
