---
title: PAM 简介与基本用法
author: tianci li
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 8.10
tags:
  - security
  - pam
---

## 文档内容说明

在本章中，您将学习关于 PAM 的以下内容：

1. 基础理论内容
2. 配置文件说明
3. 使用 PAM 增强操作系统的安全性

先决条件与假设：

* 一台非关键的 Rocky Linux PC、服务器或虚拟机
* Root 访问权限
* 一定的 Linux 基础知识
* 渴望学习 Linux 上用户和应用程序认证知识
* 能够接受自己操作带来的后果

## 介绍

PAM（**P**luggable **A**uthentication **M**odules，可插拔认证模块）是 GNU/Linux 下的系统，允许许多应用程序或服务以集中方式认证用户。PAM 最初由 Sun Microsystems（后被 Oracle 收购）的 **Vipin Samar** 和 **Charlie Lai** 于 1995 年提出，并在其 Solaris 系统上实现。后来，各种 UNIX 变体和 GNU/Linux 发行版也增加了对它的支持。PAM 设计的最初意图是将不同的底层认证机制整合到一个高级 API 中，省去开发人员自己设计和实现各种复杂认证机制的麻烦。1997 年，Open Group 发布了 X/Open Single Sign-on (XSSO) 初步规范，对 PAM API 进行了标准化，并增加了单点（或更确切地说，集成）登录的扩展。

PAM 是开源的。您可以在 [GitHub 站点上](https://github.com/linux-pam/linux-pam)找到更多信息。

* 如何确保系统中使用的应用程序、服务或工具的用户始终是他们自己？
* 如何为系统用户指定时间段以限制对服务的访问？
* 如何限制各种应用程序或服务对系统资源的使用？

没有 PAM，您只能在各种应用程序中编写认证函数。一旦需要修改特定的认证方法，开发人员可能需要重写程序、重新编译并重新安装。使用 PAM，应用程序的身份认证由 PAM 处理，因此程序主体可以不再关注身份认证本身。

![pam_image](../../guides/security/images/pam-001.png)

PAM 主要由共享库（.so 文件）和配置文件组成。其主要特性如下：

* 基于模块化设计，具有可插拔功能
* 认证方法独立于应用程序
* 为开发人员提供统一的 API
* 高度灵活性，允许应用程序通过 PAM 自由选择所需的认证方法
* 增强操作系统安全性

## PAM 中的术语

在 PAM 的早期，**Vipin Samar** 和 **Charlie Lai** 并未正式定义这些术语。然而，他们使用了未正式定义的术语，这导致使用时产生误导或难以理解的情况。1999 年，**Andrew G. Morgan**（Linux-PAM 的作者）在他的白皮书中首次建立了统一、清晰的术语，尽管当时还不完善。

FreeBSD 文档提供了以下术语的解释：

* **account（帐户）** - 申请人正在向仲裁者请求的一组凭据。
* **applicant（申请人）** - 请求认证的用户或实体。
* **arbitrator（仲裁者）** - 拥有验证申请人凭据所需权限以及授予或拒绝请求权力的用户或实体。
* **chain（链）** - 系统在响应 PAM 请求时调用一系列模块。链包括关于调用模块的顺序、传递给它们的参数以及如何解释结果的信息。
* **client（客户端）** - 负责代表申请人发起认证请求并从中获取必要认证信息的应用程序。
* **facility（设施）** - PAM 提供的四个基本功能组之一：**_authentication_**（认证）、**_account management_**（帐户管理）、**_session management_**（会话管理）和 **_authentication token update_**（认证令牌更新）。
* **module（模块）** - 实现特定认证设施的一个或多个相关函数的集合，收集到一个（通常是可动态加载的）二进制文件中，并由一个名称标识。
* **policy（策略）** - 描述如何处理特定服务的 PAM 请求的完整配置语句集。**_一个策略通常由四条链组成，每个设施一条，尽管某些服务不使用全部四个设施_**。
* **server（服务器）** - 应用程序代表仲裁者与客户端通信，检索认证信息，验证申请人的凭据，并授予或拒绝请求。
* **service（服务）** - 提供类似或相关功能并需要类似认证的一类服务器。PAM 按服务定义策略，因此所有声明相同服务名称的服务器都受同一策略约束。
* **session（会话）** - 服务器向申请人提供服务的上下文。PAM 的四个设施之一，会话管理，专门负责设置和拆除此上下文。
* **token（令牌）** - 与帐户关联的一条信息，如密码或口令短语，申请人必须提供以证明其身份。
* **transaction（事务）** - 从同一申请人到同一服务器的同一实例的一系列请求，从认证和会话设置开始，到会话拆除结束。

### 用示例说明术语

Client 和 Server 合一：

```bash
Bash > whoami
alice

Bash > ls -l `which su`

Bash > su  - 
Password: 1Q.3werzasd

Bash > whoami
root
```

* applicant（申请人）是 alice
* account（帐户）是 root
* `su` 进程既是 client 又是 server
* authentication token（认证令牌）是 1Q.3werzasd
* arbitrator（仲裁者）是 root

Client 和 Server 分离：

```bash
Bash > whoami
eve

Bash > ssh bob@login.example.com
bob@login.example.com's password:
god
Last login: Thu Oct 11 09:52:57 2024 from 192.168.0.1

```

* applicant（申请人）是 eve
* client（客户端）是 Eve 的 `ssh` 进程
* server（服务器）是 login.example.com 上的 `sshd` 进程
* account（帐户）是 bob
* authentication token（认证令牌）是 god
* 尽管在此示例中未显示，但 arbitrator（仲裁者）是 root

### 策略示例

```text
sshd	auth		required	pam_nologin.so	no_warn
sshd	auth		required	pam_unix.so	no_warn try_first_pass
sshd	account		required	pam_login_access.so
sshd	account		required	pam_unix.so
sshd	session		required	pam_lastlog.so	no_fail
sshd	password	required	pam_permit.so
```

* 此策略适用于 `sshd` 服务
* auth、account、session 和 password 是 facilities（设施）
* `pam_nologin.so`、`pam_unix.so`、`pam_login_access.so`、`pam_lastlog.so` 和 `pam_permit.so` 是 modules（模块）。从此示例可以看出，`pam_unix.so` 至少实现了两个功能组（认证和帐户管理）

## PAM 要点

### 设施和原语

计算机科学中的**原语**（Primitive）概念：由若干指令组成的进程，用于完成特定功能。系统将这些指令组合成一个在执行期间不可分割或中断的程序，确保操作的连续性和完整性。硬件或操作系统通常提供原语来实现关键的系统功能。

PAM API 提供六种不同的认证原语，分为四种类型的设施：

1. **auth** - 认证。认证申请人并建立帐户凭据。
2. **account** - 帐户管理。处理与认证无关的帐户可用性问题，例如基于时间段或服务器工作负载的访问限制。
3. **session** - 会话管理。处理与会话设置和拆除相关的任务，包括登录和记帐。
4. **password** - 密码管理。更改与帐户关联的认证令牌，可能是因为其已过期，也可能是因为用户希望更改它。

!!! tip "不同的定义"

    某些发行版将这 4 个设施定义为"模块接口"或"模块类型"。

### 模块

PAM 模块是一段独立的程序代码，实现了一个或多个设施中针对特定机制的原语。简而言之，PAM 中的模块是实现特定认证服务的一个或多个相关函数的集合。

请求 PAM 模块将返回以下三种状态之一：

1. success（成功） - 满足安全策略
2. failure（失败） - 未满足安全策略
3. ignore（忽略） - 请求未涉及安全策略

### 链和策略

当服务器发起 PAM 事务时，PAM 库会尝试为 [pam_start(3)](https://www.man7.org/linux/man-pages/man3/pam_start.3.html) 调用中指定的服务加载策略。策略指定了如何处理认证请求，如配置文件中定义的那样。这是 PAM 中的另一个核心概念：管理员只需通过编辑文本文件即可调整系统安全策略（广义上的）。

一个策略由四条链组成，每条链对应四个 PAM 设施之一。每条链是一系列配置语句，每个语句指定要调用的模块、一些传递给模块的（可选）参数，以及一个描述如何解释模块返回码的控制标志（control flag）。

#### 控制标志

这些控制标志包括：

* `binding` - 如果模块成功且链中没有更早的模块失败，则链立即终止，并授予请求。如果模块失败，则执行链的其余部分，但请求被拒绝
* `required` - 模块必须成功才能继续认证。如果测试在此点失败，用户不会收到通知，直到引用该接口的所有模块测试的结果都完成
* `requisite` - 模块必须成功才能继续认证。然而，如果测试在此点失败，用户会立即收到通知，并显示反映第一个失败的 "required" 或 "requisite" 模块测试的消息
* `optional` - 忽略模块结果。标记为 "optional" 的模块仅在没有其他模块引用该接口时才成为成功认证的必要条件
* `sufficient` - 如果模块失败，则忽略结果。然而，如果标记为 "sufficient" 的模块结果成功，且之前没有标记为 "required" 的模块失败，则不需要其他结果，用户即被认证为该服务
* `include` - 与其他控制不同，这不涉及模块结果的处理。此标志将配置文件中匹配给定参数的所有行拉入，并将其作为参数附加到模块

下图显示了如何记录每个控制标志的成功或失败：

![pam_control_flags](./images/18-pam-control-flags.jpg)

![pam_other](../../guides/security/images/pam-002.png)

对于其他控制标志，请参考 `man 5 pam.conf`。

当服务器调用六个 PAM 原语之一时，PAM 检索该原语所属设施的链，并按列出的顺序调用链中列出的每个模块，直到到达末尾，或确定不需要进一步处理（因为 "binding" 或 "sufficient" 模块成功，或因为 "requisite" 模块失败）。系统授予请求当且仅当调用了至少一个模块且所有非可选模块都成功。

!!! tip "提示"

    请注意，尽管不常见，但同一模块可能在同一个链中列出多次。例如，在目录服务器中查找用户名和密码的模块可以被多次调用，每次使用不同的参数指定要联系的不同目录服务器。PAM 将同一模块在同一链中的不同位置视为不同、不相关的模块。

## 配置文件说明

在传统的旧版本 PAM 中，配置文件是 `/etc/pam.conf`。目前，大多数发行版已放弃此配置文件。此文件包含与操作系统身份认证相关的所有策略。每行表示某个服务的某个链中的一条配置语句，其语法如下：

```bash
login   auth    required        pam_nologin.so   no_warn
```

这些字段依次是：服务名称、设施名称、控制标志、模块名称和模块参数。任何额外字段的解释都作为额外的模块参数。

> 一个策略由四条链组成，每条链对应四个 PAM 设施之一。每条链是一系列配置语句，每个语句指定要调用的模块、一些传递给模块的（可选）参数，以及一个描述如何解释模块返回码的控制标志。
> 一个策略通常由四条链组成，每个设施一条，尽管某些服务不使用全部四个设施。

OpenPAM 和 Linux-PAM 支持另一种配置机制：将策略文件集中在 **/etc/pam.d/** 目录中。在这种情况下，PAM 使用服务名称作为文件名来表示该服务的策略文件。

```bash
Bash > ls -l /etc/pam.d/
total 88
-rw-r--r--. 1 root root 232 Nov 27 03:04 config-util
-rw-r--r--. 1 root root 322 Nov 30  2023 crond
-rw-r--r--. 1 root root 701 Nov 27 03:04 fingerprint-auth
-rw-r--r--. 1 root root 715 Feb  9  2024 login
-rw-r--r--. 1 root root 154 Nov 27 03:04 other
-rw-r--r--. 1 root root 168 Apr 20  2022 passwd
-rw-r--r--. 1 root root 760 Nov 27 03:04 password-auth
-rw-r--r--  1 root root 155 May 28  2024 polkit-1
-rw-r--r--. 1 root root 398 Nov 27 03:04 postlogin
-rw-r--r--. 1 root root 640 Feb  9  2024 remote
-rw-r--r--. 1 root root 143 Feb  9  2024 runuser
-rw-r--r--. 1 root root 138 Feb  9  2024 runuser-l
-rw-r--r--  1 root root 153 Nov 27 03:04 smartcard-auth
-rw-r--r--. 1 root root 727 Aug 14 04:36 sshd
-rw-r--r--. 1 root root 214 Dec 18 01:38 sssd-shadowutils
-rw-r--r--. 1 root root 566 Feb  9  2024 su
-rw-r--r--. 1 root root 154 Feb 15  2024 sudo
-rw-r--r--. 1 root root 178 Feb 15  2024 sudo-i
-rw-r--r--. 1 root root 137 Feb  9  2024 su-l
-rw-r--r--. 1 root root 760 Nov 27 03:04 system-auth
-rw-r--r--  1 root root 368 Dec 18 01:56 systemd-user
-rw-r--r--. 1 root root  84 Jun 22  2023 vlock
```

```bash
Bash > grep -v ^# /etc/pam.d/sshd
auth       substack     password-auth
auth       include      postlogin
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
session    required     pam_selinux.so close
session    required     pam_loginuid.so
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so
session    include      password-auth
session    include      postlogin
```

每个策略文件的内容最多可以包含四个字段：

1. TYPE - auth、session、password 或 account 之一
2. CONTROL - 控制标志
3. MOUDLE_PATH - 模块路径和以 .so 结尾的模块。对于 64 位操作系统，您可以在 **/lib64/security/** 中找到所有模块
4. MOUDLE_ARGS - 控制模块行为的模块参数。可选

在 **/etc/security/** 目录中，有 PAM 模块的全局配置文件，定义其确切行为。每个使用 PAM 模块的应用程序调用一组 PAM 函数，然后这些函数处理配置文件中的信息并将结果返回给请求的应用程序。

由于 PAM 策略的确定是通过文件名而非策略文件内容指定的，因此您可以让多个服务使用相同名称的不同策略文件。例如：

```bash
Bash > cd /etc/pam.d/ && ln -s sudo ftp
```

### 策略文件内容说明

```bash
Bash > cat /etc/pam.d/system-auth
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authselect is run.
auth        required      pam_env.so
auth        sufficient    pam_unix.so try_first_pass nullok
auth        required      pam_deny.so

account     required      pam_unix.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so try_first_pass use_authtok nullok sha512 shadow
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so

Bash > cat /etc/pam.d/login
#%PAM-1.0
auth       substack     system-auth
auth       include      postlogin
account    required     pam_nologin.so
account    include      system-auth
password   include      system-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_console.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      system-auth
session    include      postlogin
-session   optional     pam_ck_connector.so
```

内容说明：

* `#%PAM-1.0` - 声明此配置文件为 PAM 1.0 版本，这是一种习惯，可用于将来检查版本。类似 bash 脚本中的 "#!/bin/bash"。
* `# This file is auto-generated.` - 以 `#` 开头的行表示注释行。
* `-session` - 前缀 "-" 表示即使模块不存在，也不会影响认证结果，更不会将此事件记录在日志中。此特性对于可选模块非常有用。
* `include` - 特殊控制标志。
* `substack` - 特殊控制标志。与 "include" 控制标志有细微差别。例如，子栈（sub-stack）中的 "requisite" 失败只会导致子栈终止并返回失败结果，而不会立即终止父栈。在 PAM 中，"stack（栈）" 指执行步骤和规则，子栈（sub-stack）指嵌套在栈（父栈）内的另一个栈。
* 从内容中可以看到，控制标志的编写风格不仅可以使用关键字（称为 "keyword" 模式），还可以使用 **[value1=action1 value2=action2 ... ]** 样式，该样式调用行模块中函数的返回码。

    * "value" 可以有：success, open_err, symbol_err, service_err, system_err, buf_err, perm_denied, auth_err, cred_insufficient, authinfo_unavail, user_unknown, maxtries, new_authtok_reqd, acct_expired, session_err, cred_unavail, cred_expired, cred_err, no_module_data, conv_err, authtok_err, authtok_recover_err, authtok_lock_busy, authtok_disable_aging, try_again, ignore, abort, authtok_expired, module_unknown, bad_item, conv_again, incomplete 和 default。
    * "action" 可以是其中之一：ignore, bad, die, ok, done, N（N 必须是大于 0 的正整数。如果等于 0，等同于 OK），reset
* `pam_unix.so` - 模块路径和模块名称。此处的路径指 **/lib64/security/** 的相对路径。
* `use_authtok nullok sha512 shadow` - 传递给模块的可选模块参数。要查看特定模块的所有可选参数，请输入 `man 8 Module-Name`，例如 `man 8 pam_unix`。

### `system-auth`

`system-auth` 是一个非常重要的 PAM 策略文件，主要负责认证用户登录系统。不仅如此，其他程序或服务可以使用 include 关键字调用它，节省重新配置的时间。当不确定如何配置时，应先搜索相关信息或咨询相关技术人员。

```bash
Bash > cat /etc/pam.d/system-auth
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authselect is run.
auth        required      pam_env.so
auth        sufficient    pam_unix.so try_first_pass nullok
auth        required      pam_deny.so

account     required      pam_unix.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so try_first_pass use_authtok nullok sha512 shadow
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
```

如您所见，此策略文件使用了完整的 4 条链，每条链对应一个设施，并且您可以堆叠每个设施（即同一设施可以写多行，每行可以调用相同或不同的模块）。再次强调：

> PAM 将同一模块在同一链中不同位置的出现视为不同且不相关的模块。

#### `auth 链`

当用户登录时，其身份和密码将通过 **auth** 验证。

* **pam_env.so** - 定义用户登录后的环境变量。默认情况下，如果没有配置文件，环境变量设置在 **/etc/security/pam_env.conf** 文件中。
* **pam_unix.so** - 使用此模块提示用户输入密码，并与 **/etc/shadow** 中记录的密码信息进行比较。如果密码比较成功，用户可以登录。使用 "sufficient" 控制标志意味着，只要此配置项的验证通过，用户就可以完全认证，而不需要请求其他模块。nullok 模块参数指示是否允许空密码。
* **pam_deny.so** - 通过 **pam_deny.so** 模块直接拒绝所有不满足上述任何条件的登录请求。**pam_deny.so** 是一个始终返回 "no" 的特殊模块。与大多数安全机制一样，不匹配认证规则的请求在所有认证规则完成后被拒绝。

#### account 链

* **pam_unix.so** - 表示用户需要通过密码验证。

#### `password 链`

* **pam_pwquality.so** - 仅在 password 设施中使用。检查密码的复杂性，常用模块参数如下：

    * debug - 启用此参数将模块的行为信息写入 syslog
    * authtok_type=XXX - 默认操作是模块在请求密码时使用以下提示："New UNIX password: " 和 "Retype UNIX password: "。您可以使用此选项替换示例词 "UNIX"。默认情况下为空。
    * try_first_pass - 表示模块应首先使用从前一个模块获取的密码，如果密码验证失败，则提示用户输入新密码。
    * local_users_only - 表示忽略不在本地 **/etc/passwd** 文件中的用户。
    * retry=N - 更改密码后允许的密码重试次数；默认值为 1。
    * minlen=N - 新密码的最小长度是多少？默认值是 8。
    * difok=N - 新旧密码之间必须不同的最小字符数是多少？默认值为 1。特殊值 0 禁用对新旧密码相似性的所有检查，但新密码与旧密码完全相同的情况除外。
    * dcredit=N - 当 N>0 时，表示新密码中允许的最大数字字符数。当 N<0 时，表示新密码中必须包含的最小数字字符数为 0。当 N=0 时，表示新密码可以包含任意数量的数字字符。
    * lcredit=N - 当 N>0 时，表示新密码中允许的最小小写字母数。当 N<0 时，表示新密码中必须包含的小写字母的最小数是 0。当 N=0 时，表示新密码中小写字母数量没有限制。
    * ucredit=N - 当 N>0 时，表示新密码中允许的最大大写字母数。当 N<0 时，表示新密码中必须包含的大写字母的最小数。当 N=0 时，表示新密码中大写字母数量没有限制。
    * ocredit=N - 当 N>0 时，表示新密码中允许的最大特殊字符数。当 N<0 时，表示新密码中必须包含的特殊字符的最小数是 0。当 N=0 时，表示新密码中特殊字符数量没有限制。
    * maxrepeat=N - 拒绝包含 N 个或更多相同连续字符的密码。默认值为 0，禁用此检查。
    * maxsequence=N - 拒绝包含长度超过 N 的单调字符序列的密码。默认值为 0，禁用此检查。此类序列的示例有 '12345' 或 'fedcb'。请注意，大多数此类密码不会通过简单性检查，除非序列只是密码的一小部分。
    * maxclassrepeat=N - 拒绝包含超过 N 个同一类别连续字符的密码。默认值为 0，禁用此检查。
    * enforce_for_root - 添加此参数后，表示密码复杂性要求也适用于 root 用户。

#### session 链

* **pam_keyinit.so** - 在用户登录期间创建相应的 keyring，并在注销后撤销它。您只能在 session 设施中使用此模块。

    * revoke - 导致在调用进程退出时撤销调用进程的会话 keyring。（前提是首先为该进程创建了会话 keyring。）

* **pam_limits.so** - 限制系统资源的模块，包括 root（uid=0）用户。默认情况下，此模块首先读取 **/etc/security/limits.conf** 文件的内容，然后读取 **/etc/security/limits.d/** 目录中的所有 .conf 文件。

* **pam_systemd.so** - `pam_systemd` 模块将在 `systemd` 登录管理器（即 `systemd-logind.service`）中注册用户会话，并在 `systemd` 控制组中注册它们。您只能在 session 设施中使用此模块。

* **pam_succeed_if.so** - 对用户帐户的特征执行逻辑判断的模块。从模块名称可以推断，您需要先定义判断条件。基本用法是 `pam_succeed_if.so  [flag...]  [condition...]`

    * 支持这些 flags：debug, use_uid, quiet, quiet_fail, quiet_success, audit。
    * Conditions 是三个词：一个字段、一个测试和要测试的值。
    * 其中，"field" 支持：user, uid, gid, shell, home, ruser, rhost, tty, service

    此模块的配置示例：

    ```text
    pam_succeed_if.so    quiet    uid < 1000  
    pam_succeed_if.so    quiet    gid  eq  1000
    pam_succeed_if.so    quiet    gid <= 1000
    pam_succeed_if.so    quiet    gid >= 1000
    pam_succeed_if.so    quiet    user = root
    pam_succeed_if.so    quiet    user   ingroup  root
    ```

## 实际配置案例

**案例 1 需求**：增加密码的复杂性。普通用户（UID >= 1000）在更改密码时的要求是：密码必须至少 10 个字符长，包含至少一个数字、至少一个大写字母、至少一个小写字母和至少一个特殊字符。当用户更改密码时，仅允许重试 3 次。

```bash
Bash > vim /etc/pam.d/system-auth
...
password requisite  pam_pwquality.so  try_first_pass  local_users_only  retry=3  authtok_type=  minlen=10  dcredit=-1  lcredit=-1  ucredit=-1  ocredit=-1
...
```

由于我没有添加 "enforce_for_root" 模块参数，密码复杂性要求不适用于 root。

```bash
Bash > whoami
root

Bash > id
uid=0(root) gid=0(root) groups=0(root)

Bash > useradd -u 3000 pam-jack
```

如果 root 用户为 pam-jack 更改密码，允许更改，尽管会有文本提示：

```bash
# 假设密码为 'qwer1234'
Bash > passwd pam-jack
Changing password for user pam-jack.
New password:
BAD PASSWORD: The password contains less than 1 uppercase letters
Retype new password:
passwd: all authentication tokens updated successfully.
```

如果 pam-jack 用户更改其密码，其密码将严格遵守复杂性要求：

```bash
Bash > su - pam-jack

# 假设密码为 "400!@GooBing."
Bash > passwd
Changing password for user pam-jack.
Current password:
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

任何不符合复杂性要求的密码都将导致接受失败：

```bash
Bash > passwd
Changing password for user pam-jack.
Current password:
New password:
BAD PASSWORD: The password contains less than 1 uppercase letters
New password:
BAD PASSWORD: The password contains less than 1 digits
New password:
BAD PASSWORD: The password contains less than 1 non-alphanumeric characters
passwd: Have exhausted maximum number of retries for service

Bash > 
```

三次失败请求后，密码更改交互过程将结束。

**案例 2 需求**：限制 SSH 密码尝试次数，达到最大限制后锁定帐户。当用户通过 SSH 远程登录时，在 900 秒内输入错误密码 3 次，结果用户帐户被锁定 180 秒。

首先，让我们看看 **pam_faillock.so** 模块的模块参数：

* preauth - 预授权。当您在其他需要用户凭据的模块（如密码）之前调用该模块时，必须使用 preauth 参数。该模块仅检查是否应阻止用户访问该服务，如果最近有异常数量的连续认证失败尝试。
* authfail - 当确定认证结果的模块失败后调用该模块时，必须使用 authfail 参数。
* authsucc - 当确定认证结果的模块成功后调用该模块时，必须使用 authsucc 参数。
* silent - 不打印各种类型的消息。
* deny=n - 如果尝试登录的用户在最近的间隔内连续登录失败超过 n 次，则拒绝用户访问。默认值为 3。
* audit - 如果不存在相应的用户，则在系统日志中记录用户名。
* even_deny_root - 锁定帐户的限制也适用于 root。
* root_unlock_time=n - 设置锁定 root 帐户的时间。如果未指定此参数，则使用 unlock_time 参数的值代替。
* unlock_time=n - 锁定帐户的时间单位是秒，默认值为 600。超过此时间限制后，帐户将自动解锁。
* fail_interval=n - 定义登录失败之间的间隔，默认值为 900。也就是说，它统计 15 分钟内的登录失败次数。

以上参数仅为简要说明。更详细的信息，请参考 `man 8 pam_faillock`。

默认情况下，在不更改配置文件的情况下，SSH 服务器允许用户在登录时重试 6 次，而 SSH 客户端允许 3 次重试。您可以通过手册页查看：

```bash
Bash > man 5 sshd_config
...
MaxAuthTries
            Specifies the maximum number of authentication attempts permitted per connection.  Once the number of failures reaches half this value, additional failures are logged. The default is 6.
...

Bash > man 5 ssh_config
...
NumberOfPasswordPrompts
            Specifies the number of password prompts before giving up.  The argument to this keyword must be an integer. The default is 3.
```

在 PowerShell 中指定客户端尝试次数：

```text
PS > ssh -p 22 -o NumberOfPasswordPrompts=6 root@192.168.100.20
```

sshd 策略文件包含 password-auth 文件的内容：

```bash
Bash > cat /etc/pam.d/sshd
...
account    include      password-auth
...
```

修改 password-auth 文件的内容：

```bash
Bash > vim /etc/pam.d/password-auth
auth        required      pam_faillock.so  preauth  silent audit even_deny_root deny=3  unlock_time=180  <<< 添加新行
auth        required      pam_env.so
auth        sufficient    pam_unix.so try_first_pass nullok
auth      [default=die]   pam_faillock.so  authfail  audit  even_deny_root  deny=3 unlock_time=180  <<< 添加新行
auth        required      pam_deny.so

account     required      pam_unix.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=5 authtok_type=
password    sufficient    pam_unix.so try_first_pass use_authtok nullok sha512 shadow
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
```

SSH 客户端连续三次故意输入错误密码：

```text
PS > ssh -p 22 root@192.168.100.20
root@192.168.100.20's password:
Permission denied, please try again.
root@192.168.100.20's password:
Permission denied, please try again.
root@192.168.100.20's password:
root@192.168.100.20: Permission denied (publickey,gssapi-keyex,gssapi-with-mic, password).
```

重新连接时，即使您的密码正确，也不会通过身份认证，因为帐户已被锁定 180 秒。

您可以在 SSH 服务器中使用 `faillock` 命令查看具体信息。您可以在 **/var/log/secure** 日志中查看锁定信息。

```bash
Bash > faillock
root:
When                Type  Source                                           Valid
2025-03-31 20:15:19 RHOST 192.168.100.8                                        V
2025-03-31 20:15:55 RHOST 192.168.100.8                                        V
2025-03-31 20:15:59 RHOST 192.168.100.8                                        V

Bash > grep -i lock /var/log/secure
Mar 31 20:15:59 HOME01 sshd[6239]: pam_faillock(sshd:auth): Consecutive login failures for user root account temporarily locked
```

您可以在 **/var/run/faillock/** 目录中找到登录失败尝试后的相关帐户数据。要清除数据，使用 `faillock --reset` 命令。

**案例 3 需求**：禁止将密码更改为最近三次使用过的密码。

只需使用 `pam_pwhistory.so` 模块的 "remember" 模块参数。

!!! tip "提示"

    只有在用户密码成功更改后，PAM 才能将其定义为"已记住的密码"。用户密码的成功更改将记录在 **/etc/security/opasswd** 文件中。

```bash
Bash > vim /etc/pam.d/system-auth
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authselect is run.
auth        required      pam_env.so
auth        sufficient    pam_unix.so try_first_pass nullok
auth        required      pam_deny.so

account     required      pam_unix.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    requisite     pam_pwhistory.so  remember=3 use_authtok  <<<< 添加新行
password    sufficient    pam_unix.so try_first_pass use_authtok nullok sha512 shadow
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
```

```bash
Bash > useradd -u 3000 hisuser

# 使用 root 用户首次更改 hisuser 用户的密码
## 密码为 Flzx3QC<...>
Bash > passwd hisuser

# 第二次密码更改
## 密码为 Talk---5Ing,
Bash > passwd hisuser

# 第三次密码更改
## 密码为 u>=1000And
Bash > passwd hisuser

Bash > cat /etc/security/opasswd.old
hisuser:2500:2:!!,$6$boNLeNZGXelC2G6H$zlLzz3IMMkUeLozLNRN5OljyGHiNacWKNX.6j.RUEb1mI.pgQH7TubBO8QRCEW7ZcyKM6boTE3CUZKL2ybKDw0

Bash > cat /etc/security/opasswd
hisuser:2500:3:!!,$6$boNLeNZGXelC2G6H$zlLzz3IMMkUeLozLNRN5OljyGHiNacWKNX.6j.RUEb1mI.pgQH7TubBO8QRCEW7ZcyKM6boTE3CUZKL2ybKDw0,$6$8Ohz/v429eBRjk58$N7lG7E9sNYAhD1xLSi0S33teWFuf2udEc5ECwqBiV2x2314s6tt2eZu8EZiVAitIIzeYM9zMJtREOpNQLj05Q.
```

文档内容说明：

* 使用 ":" 分隔 4 个字段
* 这四个字段从左到右分别是：用户名、uid、记录旧密码的数量、密码密文
* 在第四个字段中，使用 "," 分隔并记录历史密码密文

如果普通用户（uid>=1000）将其密码更改为最近三次使用过的密码之一，将输出以下文本内容：

```text
Password has been already used. Choose another.
```

## 其他模块简要描述

* **pam_cracklib.so** - 检查密码是否包含字典单词的 PAM 模块
* **pam_time.so** - 用于时间控制访问的 PAM 模块
* **pam_nologin.so** - 防止非 root 用户登录
* **pam_wheel.so** - 仅允许 wheel 组成员 root 访问

## 使用提示

请注意相应模块适用于哪些设施。

```bash
Bash > man 8 pam_unix
...
MODULE TYPES PROVIDED
       All module types (account, auth, password, and session) are provided.
...

Bash > man 8 pam_success_if
...
MODULE TYPES PROVIDED
       All module types (account, auth, password, and session) are provided.
...

Bash > man 8 pam_faillock
...
MODULE TYPES PROVIDED
       The auth and account module types are provided.
...
```

至此，您应该对 PAM 能做什么以及需要时如何更改有了更好的了解。但是，在更改 PAM 模块时请务必非常小心。我们强烈建议在测试 PAM 模块相应功能之前对操作系统进行快照。
