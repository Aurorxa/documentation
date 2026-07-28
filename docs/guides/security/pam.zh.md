---
title: PAM 认证模块
author: Antoine Le Morvan
contributors: Steven Spencer, Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.5, 8.6
tags:
  - security
  - pam
---

# PAM 认证模块

## 前提条件与假设

* 一台非关键任务的 Rocky Linux PC、服务器或 VM
* Root 访问权限
* 一些现有的 Linux 知识（将有很大帮助）
* 学习关于 Linux 上用户和应用认证的愿望
* 能够接受自己行为后果的能力

## 简介

PAM（**Pluggable Authentication Modules，可插拔认证模块**）是 GNU/Linux 下允许许多应用程序或服务以集中方式认证用户的系统。换句话说：

> PAM 是一个库套件，允许 Linux 系统管理员配置认证用户的方法。它提供了一种灵活和集中的方式，使用配置文件而不是更改应用程序代码来切换安全应用程序的认证方法。 
> \- [Wikipedia](https://en.wikipedia.org/wiki/Linux_PAM)

本文档*不*旨在教您如何精确地加固您的机器。它更像一个参考指南，向您展示 PAM *可以*做什么，而不是您*应该*做什么。

## 概述

认证是在此期间验证您是否是您声称的那个人的阶段。最常见的示例是密码，但存在其他形式的认证。

![PAM 概述](images/pam-001.png)

实现新的认证方法不应该需要更改程序或服务配置的源代码。这就是为什么应用程序依赖 PAM，它为它们提供了认证用户所需的基本原语*。

系统中的所有应用因此可以以一种完全透明的方式实现复杂的功能，如 **SSO**（Single Sign On，单点登录）、**OTP**（One Time Password，一次性密码）或 **Kerberos**。系统管理员可以精确选择针对单个应用程序（例如加固 SSH 服务）要使用的认证策略，独立于该应用程序。

每个支持 PAM 的应用程序或服务将在 `/etc/pam.d/` 目录中有一个对应的配置文件。例如，进程 `login` 将名称 `/etc/pam.d/login` 分配给其配置文件。

\* 原语（Primitives）字义上是一个程序或语言中最简单的元素，允许您在此基础上构建更复杂和精妙的东西。

!!! WARNING

    配置错误的 PAM 实例可能危及整个系统的安全。如果 PAM 是脆弱的，那么整个系统就是脆弱的。请谨慎进行任何更改。

## 指令

指令用于为 PAM 使用设置应用程序。指令将遵循以下格式：

```
mechanism [control] path-to-module [argument]
```

一个**指令**（一行完整内容）由一个**机制**（`auth`、`account`、`password` 或 `session`）、一个**成功检查**（`include`、`optional`、`required` 等）、**模块路径**和可能的**参数**（例如 `revoke`）组成。

每个 PAM 配置文件包含一组指令。模块接口指令可以堆叠或逐层放置。事实上，**模块列出的顺序对认证过程非常重要。**

例如，这里是配置文件 `/etc/pam.d/sudo`：

```
#%PAM-1.0
auth       include      system-auth
account    include      system-auth
password   include      system-auth
session    include      system-auth
```

## 机制

### `auth` - 认证

这处理请求者的认证并建立账户的权限：

* 通常通过将密码与存储在数据库中的值进行比较，或依赖认证服务器进行认证，

* 建立账户设置：uid、gid、组和资源限制。

### `account` - 账户管理

检查请求的账户是否可用：

* 涉及出于认证之外的原因的账户可用性（例如时间限制）。

### `session` - 会话管理

涉及会话设置和终止：

* 执行与会话设置相关的任务（例如日志记录），
* 执行与会话终止相关的任务。

### `password` - 密码管理

用于修改与账户关联的认证令牌（过期或更改）：

* 更改认证令牌，并可能验证其是否足够强大或是否尚未被使用。

## 控制指标

PAM 机制（`auth`、`account`、`session` 和 `password`）指示 `success` 或 `failure`。控制标志（`required`、`requisite`、`sufficient`、`optional`）告诉 PAM 如何处理此结果。

### `required`

所有 `required` 模块的成功完成是必要的。

* **如果模块通过：** 链的其余部分被执行。请求被允许，除非其他模块失败。

* **如果模块失败：** 链的其余部分被执行。最终请求被拒绝。

模块必须成功验证才能继续认证。如果标记为 `required` 的模块验证失败，在所有与该接口关联的模块被验证之前不会通知用户。

### `requisite`

所有 `requisite` 模块的成功完成是必要的。

* **如果模块通过：** 链的其余部分被执行。请求被允许，除非其他模块失败。

* **如果模块失败：** 请求立即被拒绝。

模块必须成功验证才能继续认证。然而，如果标记为 `requisite` 的模块验证失败，用户会立即被通知第一个 `required` 或 `requisite` 模块失败的消息。

### `sufficient`

标记为 `sufficient` 的模块可用于在某些条件下让用户"提前"进入：

* **如果模块成功：** 如果之前的模块没有失败，认证请求立即被允许。

* **如果模块失败：** 该模块被忽略。链的其余部分被执行。

然而，如果标记为 `sufficient` 的模块检查成功，但标记为 `required` 或 `requisite` 的模块检查失败，则 `sufficient` 模块的成功被忽略，请求失败。

### `optional`

模块被执行，但请求的结果被忽略。如果链中的所有模块都标记为 `optional`，所有请求将始终被接受。

### 结论

![Rocky Linux 安装启动画面](images/pam-002.png)

## PAM 模块

PAM 有许多模块。以下是最常见的：

* pam_unix
* pam_ldap
* pam_wheel
* pam_cracklib
* pam_console
* pam_tally
* pam_securetty
* pam_nologin
* pam_limits
* pam_time
* pam_access

### `pam_unix`

`pam_unix` 模块允许您管理全局认证策略。

在 `/etc/pam.d/system-auth` 中您可以添加：

```
password sufficient pam_unix.so sha512 nullok
```

此模块可能的参数：

* `nullok`：在 `auth` 机制中允许空登录密码。
* `sha512`：在 password 机制中，定义加密算法。
* `debug`：向 `syslog` 发送信息。
* `remember=n`：使用此选项记住最后 `n` 个使用过的密码（与 `/etc/security/opasswd` 文件结合使用，该文件由管理员创建）。

### `pam_cracklib`

`pam_cracklib` 模块允许您测试密码。

在 `/etc/pam.d/password-auth` 中添加：

```
password sufficient pam_cracklib.so retry=2
```

此模块使用 `cracklib` 库来检查新密码的强度。它还可以检查新密码是否不是从旧密码构建的。它*仅*影响 password 机制。

默认情况下此模块检查以下方面，如果是这种情况则拒绝：

* 新密码是字典中的吗？
* 新密码是旧密码的回文吗（例如：azerty <> ytreza）？
* 用户只更改了密码大小写吗（例如：azerty <>AzErTy）？

此模块可能的参数：

* `retry=n`：施加 `n` 次请求（默认为 1 次）以输入新密码。
* `difok=n`：施加至少 `n` 个字符（默认为 `10`）与旧密码不同。如果新密码的一半字符与旧密码不同，则新密码被验证。
* `minlen=n`：施加最小 `n+1` 个字符的密码。您不能分配低于 6 个字符的最小值（模块是这样编译的）。

其他可能的参数：

* `dcredit=-n`：施加至少包含 `n` 个数字的密码，
* `ucredit=-n`：施加至少包含 `n` 个大写字母的密码，
* `credit=-n`：施加至少包含 `n` 个小写字母的密码，
* `ocredit=-n`：施加至少包含 `n` 个特殊字符的密码。

### `pam_tally`

`pam_tally` 模块允许您基于不成功的登录尝试次数锁定账户。

此模块的默认配置文件可能如下：`/etc/pam.d/system-auth`：

```
auth required /lib/security/pam_tally.so onerr=fail no_magic_root
account required /lib/security/pam_tally.so deny=3 reset no_magic_root
```

`auth` 机制接受或拒绝认证并重置计数器。

`account` 机制递增计数器。

pam_tally 模块的一些参数包括：

* `onerr=fail`：递增计数器。
* `deny=n`：一旦超过 `n` 次不成功尝试，账户被锁定。
* `no_magic_root`：可用于拒绝对由守护进程启动的 root 级别服务的访问。 
    * 例如不要为 `su` 使用此选项。
* `reset`：如果认证被验证，将计数器重置为 0。
* `lock_time=nsec`：账户被锁定 `n` 秒。

此模块与不成功尝试的默认文件 `/var/log/faillog`（可以用参数 `file=xxxx` 替换为另一个文件）和关联的命令 `faillog` 一起工作。

faillog 命令的语法：

```
faillog[-m n] |-u login][-r]
```

选项：

* `m`：在命令显示中定义不成功尝试的最大次数，
* `u`：指定一个用户，
* `r`：解锁一个用户。

### `pam_time`

`pam_time` 模块允许您限制对 PAM 管理服务的访问时间。

要激活它，编辑 `/etc/pam.d/system-auth` 并添加：

```
account required /lib/security/pam_time.so
```

配置在 `/etc/security/time.conf` 文件中完成：

```
login ; * ; users ;MoTuWeThFr0800-2000
http ; * ; users ;Al0000-2400
```

指令的语法如下：

```
services; ttys; users; times
```

在以下定义中，逻辑列表使用：

* `&`：是"and"逻辑。
* `|`：是"or"逻辑。
* `!`：表示否定，或"除了所有"。
* `*`：是通配符。

各列对应：

* `services`：由 PAM 管理的服务的逻辑列表，这些服务也需要由此规则管理
* `ttys`：相关设备的逻辑列表
* `users`：由规则管理的用户的逻辑列表
* `times`：授权时间段的逻辑列表

如何管理时间段：

* 天数：`Mo`、`Tu`、`We`、`Th`、`Fr`、`Sa`、`Su`、`Wk`（周一到周五）、`Wd`（周六和周日）和 `Al`（周一到周日）
* 小时范围：`HHMM-HHMM`
* 重复取消效果：`WkMo` = 一周的所有天（M-F），减去周一（重复）。

示例：

* Bob 可以每天在 07:00 到 09:00 之间通过终端登录，除了周三：

```
login; tty*; bob; alth0700-0900
```

不能登录，终端或远程，除了 root，一周的每天在 17:30 到第二天 7:45 之间：

```
login; tty* | pts/*; !root; !wk1730-0745
```

### `pam_nologin`

`pam_nologin` 模块禁用除 root 之外的所有账户：

在 `/etc/pam.d/login` 中您放置：

```
auth required pam_nologin.so
```

仅当文件 `/etc/nologin` 存在且可读时，root 可以连接。

### `pam_wheel`

`pam_wheel` 模块允许您限制对 `su` 命令的访问仅限于 `wheel` 组的成员。

在 `/etc/pam.d/su` 中您放置：

```
auth required pam_wheel.so
```

参数 `group=my_group` 将对 `su` 命令的使用限制为 `my_group` 组的成员

!!! NOTE

    如果组 `my_group` 为空，那么 `su` 命令在系统上不再可用，这强制使用 sudo 命令。

### `pam_mount`

`pam_mount` 模块允许您为用户会话挂载卷。

在 `/etc/pam.d/system-auth` 中您放置：

```
auth optional pam_mount.so
password optional pam_mount.so
session optional pam_mount.so
```

挂载点在 `/etc/security/pam_mount.conf` 文件中配置：

```
<volume fstype="nfs" server="srv" path="/home/%(USER)" mountpoint="~" />
<volume user="bob" fstype="smbfs" server="filesrv" path="public" mountpoint="/public" />
```

## 总结

到现在为止，您应该对 PAM 能做什么以及如何在需要时进行更改有了更好的理解。然而，我们必须重申，在您对 PAM 模块进行任何更改时要非常、*非常*小心。您可能会将自己锁定在系统之外，或者更糟，让其他人都进来。

我们强烈建议在可以轻松恢复到先前配置的环境中进行测试。话虽如此，尽情使用吧！
