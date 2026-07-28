---
title: 基于 Web 的应用程序防火墙 (WAF)
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - web
  - security
  - apache
  - nginx
---
  
!!! Note "尚未测试"

    此步骤可能可以按原样工作。截至 2025 年 9 月 22 日，在 Rocky Linux 10 上的测试尚未完成。如果你使用它并发现问题，请告诉我们。

## 前提条件

* 一台运行 Apache 的 Rocky Linux Web 服务器
* 熟练使用命令行编辑器（本示例使用 _vi_）
* 对在命令行中执行命令、查看日志以及其他一般系统管理员职责有较高的熟练度
* 理解安装此工具还需要监控操作并根据你的环境进行调优
* 所有命令由 root 用户或具有 `sudo` 权限的普通用户运行

## 简介

`mod_security` 是一个开源的基于 Web 的应用程序防火墙 (WAF)。它只是一个加固 Apache Web 服务器设置中可能的一个组成部分。你可以单独使用它，也可以与其他工具一起使用。

如果你想使用此工具和其他加固工具，请参考 [Apache 加固 Web 服务器指南](index.md)。本文档也使用了该原始文档中概述的所有假设和约定。在继续之前，建议你查看它。

从 Atomicorp 仓库安装 `mod_security` 时，一个缺失的问题是安装的规则是最小化的。为了获得更广泛的无成本 `mod_security` 规则包，本步骤使用[本文档中的 OWASP `mod_security` 规则](https://coreruleset.org/)。OWASP 代表开放 Web 应用程序安全项目 (Open Web Application Security Project)。你可以在[此处了解有关 OWASP 的更多信息](https://owasp.org/)。

!!! tip

    如前所述，本步骤使用 OWASP `mod_security` 规则。未使用的是该网站提供的配置。该网站还提供了使用 `mod_security` 和其他安全相关工具的精彩教程。你现在正在阅读的文档仅帮助你在 Rocky Linux Web 服务器上安装使用 `mod_security` 进行加固所需的工具和规则。Netnea 是一个技术专业人员团队，在其网站上提供安全课程。这些内容大多免费提供，但他们 *确实* 有现场培训或团体培训的选项。

## 额外仓库

要安装 `mod_security`，你需要 Atomicorp 仓库 (atomic.repo)。使用以下命令进行安装，并对所有默认选项回答 yes：

```bash
wget -q -O - https://www.atomicorp.com/installers/atomic | sh
```

运行 `dnf upgrade` 以读取所有更改。

## 安装 `mod_security`

要安装基础软件包，使用以下命令。它将安装所有缺失的依赖项。如果你还没有安装 `wget`，也需要安装它：

```bash
dnf install mod_security wget
```

## 安装 `mod_security` 规则

!!! note

    仔细按照此步骤操作非常重要。来自 Netnea 的配置已经过修改以适应 Rocky Linux。

1. 通过[访问他们的 GitHub 站点](https://github.com/coreruleset/coreruleset)来获取当前的 OWASP 规则。

2. 在页面右侧，搜索发布版本并点击最新版本的标签。

3. 在下一页的 "Assets" (资产) 下，右键点击 "Source Code (tar.gz)" 链接并复制链接。

4. 在你的服务器上，进入 Apache 配置目录：

    ```bash
    cd /etc/httpd/conf
    ```

5. 输入 `wget` 并粘贴你的链接。示例：

    ```bash
    wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.5.tar.gz
    ```

6. 解压文件：

    ```bash
    tar xzvf v3.3.5.tar.gz
    ```

    这将创建一个名称中包含发布信息的目录。示例："coreruleset-3.3.5"

7. 创建一个名为 "crs" 的符号链接，链接到发布版本的目录。示例：

    ```bash
    ln -s coreruleset-3.3.5/ /etc/httpd/conf/crs
    ```

8. 删除 `tar.gz` 文件。示例：

    ```bash
    rm -f v3.3.5.tar.gz
    ```

9. 复制临时配置以便在启动时加载：

    ```bash
    cp crs/crs-setup.conf.example crs/crs-setup.conf
    ```

    这个文件是可编辑的，但你很可能不需要做任何更改。

`mod_security` 规则现在已经就位。

## 配置

规则就位后，下一步是配置这些规则，使其在 `httpd` 和 `mod_security` 运行时加载并执行。

`mod_security` 已经有一个配置文件位于 `/etc/httpd/conf.d/mod_security.conf`。你需要修改此文件以包含 OWASP 规则。为此，编辑该配置文件：

```bash
vi /etc/httpd/conf.d/mod_security.conf
```

在结束标签 (`</IfModule`) 之前添加以下内容：

```bash
    Include    /etc/httpd/conf/crs/crs-setup.conf

    SecAction "id:900110,phase:1,pass,nolog,\
        setvar:tx.inbound_anomaly_score_threshold=10000,\
        setvar:tx.outbound_anomaly_score_threshold=10000"

    SecAction "id:900000,phase:1,pass,nolog,\
         setvar:tx.paranoia_level=1"


    # === ModSec Core Rule Set: Runtime Exclusion Rules (ids: 10000-49999)

    # ...


    # === ModSecurity Core Rule Set Inclusion

    Include    /etc/httpd/conf/crs/rules/*.conf


    # === ModSec Core Rule Set: Startup Time Rules Exclusions

    # ...
```

使用 ++esc++ 退出插入模式，然后使用 ++shift+colon+"wq"++ 保存更改并退出。

## 重启 `httpd` 并验证 `mod_security`

此时你只需要重启 `httpd`：

```bash
systemctl restart httpd
```

验证服务是否如预期启动：

```bash
systemctl status httpd
```

`/var/log/httpd/error_log` 中类似以下的条目将表明 `mod_security` 正在正确加载：

```bash
[Thu Jun 08 20:31:50.259935 2023] [:notice] [pid 1971:tid 1971] ModSecurity: PCRE compiled version="8.44 "; loaded version="8.44 2020-02-12"
[Thu Jun 08 20:31:50.259936 2023] [:notice] [pid 1971:tid 1971] ModSecurity: LUA compiled version="Lua 5.4"
[Thu Jun 08 20:31:50.259937 2023] [:notice] [pid 1971:tid 1971] ModSecurity: YAJL compiled version="2.1.0"
[Thu Jun 08 20:31:50.259939 2023] [:notice] [pid 1971:tid 1971] ModSecurity: LIBXML compiled version="2.9.13"
```

如果你访问服务器上的 Web 站点，你应该会在 `/var/log/httpd/modsec_audit.log` 中收到一条显示 OWASP 规则已加载的条目：

```bash
Apache-Handler: proxy:unix:/run/php-fpm/www.sock|fcgi://localhost
Stopwatch: 1686249687051191 2023 (- - -)
Stopwatch2: 1686249687051191 2023; combined=697, p1=145, p2=458, p3=14, p4=45, p5=35, sr=22, sw=0, l=0, gc=0
Response-Body-Transformed: Dechunked
Producer: ModSecurity for Apache/2.9.6 (http://www.modsecurity.org/); OWASP_CRS/3.3.4.
Server: Apache/2.4.53 (Rocky Linux)
Engine-Mode: "ENABLED"
```

## 结论

带有 OWASP 规则的 `mod_security` 是加固 Apache Web 服务器的另一个工具。定期检查 [GitHub 站点上的新规则](https://github.com/coreruleset/coreruleset)和最新正式版本是你要做的持续维护步骤。

`mod_security` 和其他加固工具一样，有可能产生误报，因此你必须准备好为你的安装调优此工具。

像 [Apache 加固 Web 服务器指南](index.md)中提到的其他解决方案一样，`mod_security` 规则还有其他免费和付费的解决方案，而且还有其他可用的 WAF 应用程序。你可以在 [Atomicorp 的 `mod_security` 站点](https://atomicorp.com/atomic-modsecurity-rules/)查看其中之一。
