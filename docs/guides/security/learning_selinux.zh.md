---
title: SELinux 安全
author: Antoine Le Morvan
contributors: Steven Spencer, markooff, Ganna Zhyrnova
tags:
  - security
  - SELinux
---

# SELinux 安全

随着内核 2.6 版本的到来，引入了一个新的安全系统，以提供支持访问控制安全策略的安全机制。

此系统称为 **SELinux**（**S**ecurity **E**nhanced **Linux，安全增强型 Linux**），由 **NSA**（**N**ational **S**ecurity **A**gency，美国国家安全局）创建，以在 Linux 内核子系统中实现强大的 **M**andatory **A**ccess **C**ontrol（**MAC，强制访问控制**）架构。

如果在您的职业生涯中，您要么禁用了 SELinux 要么忽略了它，本文档将是对此系统的一个良好介绍。SELinux 致力于限制权限或消除与程序或守护进程被攻陷相关的风险。

在开始之前，您应该知道 SELinux 主要用于 RHEL 发行版，尽管也可能在其他发行版（如 Debian）上实现它（但祝您好运！）。Debian 系列的发行版通常集成了 AppArmor 系统，其工作方式与 SELinux 不同。

## 概述

**SELinux**（Security Enhanced Linux，安全增强型 Linux）是一个强制访问控制系统。

在 MAC 系统出现之前，标准访问管理安全基于 **DAC**（**D**iscretionary **A**ccess **C**ontrol，自主访问控制）系统。一个应用程序或守护进程以 **UID** 或 **SUID**（**S**et **O**wner **U**ser **Id，设置所有者用户 ID）权限运行，这使得可以根据此用户评估权限（对文件、套接字和其他进程...）。这种操作没有充分限制被破坏程序的权限，可能使其能够访问操作系统的子系统。

MAC 系统加强了机密性和完整性信息的分离，以实现一个遏制系统。遏制系统独立于传统权限系统，没有超级用户的概念。

每次系统调用时，内核都会查询 SELinux 是否允许执行该操作。

![SELinux](../images/selinux_001.png)

SELinux 使用一组规则（策略）来实现此目的。提供了两组标准规则集（**targeted** 和 **strict**），每个应用程序通常提供其自己的规则。

### SELinux 上下文

SELinux 的操作方式与传统的 Unix 权限完全不同。

SELinux 安全上下文由 **identity**（身份）+**role**（角色）+**domain**（域）三要素定义。

用户的身份直接取决于其 Linux 账户。一个身份被分配一个或多个角色，但每个角色对应一个域，且仅对应一个。

根据安全上下文的域（因此也是角色）来评估用户对资源的权限。

![SELinux 上下文](../images/selinux_002.png)

"domain"（域）和 "type"（类型）这两个术语是相似的。通常 "domain" 指进程，而 "type" 指对象。

命名约定是：**user_u:role_r:type_t**。

安全上下文在用户连接期间根据其角色分配给用户。文件的安全上下文由 `chcon`（**ch**ange **con**text）命令定义，我们将在本文档后面看到。

考虑 SELinux 拼图的以下部分：

- 主体（subjects）
- 客体（objects）
- 策略（policies）
- 模式（mode）

当主体（例如应用程序）尝试访问客体（例如文件）时，Linux 内核的 SELinux 部分查询其策略数据库。根据操作模式，SELinux 在成功时授权访问客体，否则在文件 `/var/log/messages` 中记录失败。

#### 标准进程的 SELinux 上下文

进程的权限取决于其安全上下文。

默认情况下，进程的安全上下文由启动它的用户的上下文（身份 + 角色 + 域）定义。

域是与进程相关并从启动它的用户继承（通常）的特定类型（在 SELinux 意义上）。其权限以对与客体相关的类型的授权或拒绝来表示：

具有安全**域 D** 的进程可以访问**类型 T** 的客体。

![标准进程的 SELinux 上下文](../images/selinux_003.png)

#### 重要进程的 SELinux 上下文

大多数重要程序都被分配了专用域。

每个可执行文件被标记为专用类型（此处为 **sshd_exec_t**），该类型自动将相关进程切换到 **sshd_t** 上下文（而不是 **user_t**）。

此机制至关重要，因为它尽可能限制了进程的权限。

![重要进程的 SELinux 上下文 - sshd 示例](../images/selinux_004.png)

## 管理

`semanage` 命令管理 SELinux 规则。

```bash
semanage [object_type] [options]
```

示例：

```bash
semanage boolean -l
```

| 选项 | 说明      |
|---------|-------------------|
| -a      |  添加一个对象   |
| -d      |  删除一个对象 |
| -m      |  修改一个对象 |
| -l      |  列出对象 |

`semanage` 命令在 Rocky Linux 下可能默认未安装。

在不知道提供此命令的软件包的情况下，您应该使用以下命令搜索其名称：

```bash
dnf provides */semanage
```

然后安装它：

```bash
sudo dnf install policycoreutils-python-utils
```

### 管理布尔对象

布尔值允许对进程进行遏制。

```bash
semanage boolean [options]
```

列出可用的布尔值：

```bash
semanage boolean –l
SELinux boolean    State Default  Description
…
httpd_can_sendmail (off , off)  Allow httpd to send mail
…
```

!!! Note

    正如您所见，有一个 `default` 状态（例如在启动时）和一个运行状态。

`setsebool` 命令用于更改布尔对象的状态：

```bash
setsebool [-PV] boolean on|off
```

示例：

```bash
sudo setsebool -P httpd_can_sendmail on
```

| 选项 | 说明                                                       |
|---------|--------------------------------------------------------------------|
| `-P`    | 更改启动时的默认值（否则仅持续到重启） |
| `-V`    | 删除一个对象                                                  |

!!! Warning

    不要忘记使用 `-P` 选项以在下次启动后保持状态。

### 管理端口对象

`semanage` 命令用于管理端口类型的对象：

```bash
semanage port [options]
```

示例：允许端口 81 用于 httpd 域进程

```bash
sudo semanage port -a -t http_port_t -p tcp 81
```

## 操作模式

SELinux 有三种操作模式：

- Enforcing（强制模式）

Rocky Linux 的默认模式。访问将根据生效的规则受到限制。

- Permissive（宽容模式）

规则被查询，访问错误被记录，但访问不会被阻止。

- Disabled（禁用）

不会限制任何内容，不会记录任何内容。

默认情况下，大多数操作系统配置 SELinux 为 Enforcing 模式。

`getenforce` 命令返回当前操作模式

```bash
getenforce
```

示例：

```bash
$ getenforce
Enforcing
```

`sestatus` 命令返回有关 SELinux 的信息

```bash
sestatus
```

示例：

```bash
$ sestatus
SELinux status:       enabled
SELinuxfs mount:     /sys/fs/selinux
SELinux root directory:    /etc/selinux
Loaded policy name:        targeted
Current mode:             enforcing
Mode from config file:     enforcing
...
Max kernel policy version: 33
```

`setenforce` 命令更改当前操作模式：

```bash
setenforce 0|1
```

将 SELinux 切换到 permissive 模式：

```bash
sudo setenforce 0
```

### `/etc/sysconfig/selinux` 文件

`/etc/sysconfig/selinux` 文件允许您更改 SELinux 的操作模式。

!!! Warning

    禁用 SELinux 风险自负！最好学习 SELinux 的工作方式，而不是系统性地禁用它！

编辑文件 `/etc/sysconfig/selinux`

```bash
SELINUX=disabled
```

!!! Note

    `/etc/sysconfig/selinux` 是指向 `/etc/selinux/config` 的符号链接

重启系统：

```bash
sudo reboot
```

!!! Warning

    注意 SELinux 模式更改！

在 disabled 模式下，新创建的文件将没有任何标签。

要重新激活 SELinux，您将必须在整个系统上重新设置标签。

为整个系统打标签：

```bash
sudo touch /.autorelabel
sudo reboot
```

## 策略类型

SELinux 提供了两种标准类型的规则：

- **Targeted（目标型）**：仅网络守护进程受保护（`dhcpd`、`httpd`、`named`、`nscd`、`ntpd`、`portmap`、`snmpd`、`squid` 和 `syslogd`）
- **Strict（严格型）**：所有守护进程都受保护

## 上下文

安全上下文的显示使用 `-Z` 选项完成。它与许多命令关联：

示例：

```bash
id -Z # 用户的上下文
ls -Z # 当前文件的上下文
ps -eZ # 进程的上下文
netstat –Z # 网络连接
lsof -Z # 打开的文件
```

`matchpathcon` 命令返回目录的上下文。

```bash
matchpathcon directory
```

示例：

```bash
sudo matchpathcon /root
 /root system_u:object_r:admin_home_t:s0

sudo matchpathcon /
 /      system_u:object_r:root_t:s0
```

`chcon` 命令修改安全上下文：

```bash
chcon [-vR] [-u USER] [–r ROLE] [-t TYPE] file
```

示例：

```bash
sudo chcon -vR -t httpd_sys_content_t /data/websites/
```

| 选项        | 说明                    |
|----------------|---------------------------------|
| `-v`           | 切换到详细模式        |
| `-R`           | 应用递归                 |
| `-u`,`-r`,`-t` | 应用于用户、角色或类型 |

`restorecon` 命令恢复默认安全上下文（规则提供的那个）：

```bash
restorecon [-vR] directory
```

示例：

```bash
sudo restorecon -vR /home/
```

| 选项 | 说明             |
|---------|--------------------------|
| `-v`    | 切换到详细模式 |
| `-R`    | 应用递归          |

要使上下文更改在 `restorecon` 后持久化，您必须使用 `semanage fcontext` 命令修改默认文件上下文：

```bash
semanage fcontext -a options file
```

!!! Note

    如果您正在为系统非标准的文件夹执行上下文切换，创建规则然后应用上下文是一个好做法，如下例所示！

示例：

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/data/websites(/.*)?"
sudo restorecon -vR /data/websites/
```

## `audit2why` 命令

`audit2why` 命令指出 SELinux 拒绝的原因：

```bash
audit2why [-vw]
```

获取 SELinux 最后一次拒绝的原因的示例：

```bash
sudo cat /var/log/audit/audit.log | grep AVC | grep denied | tail -1 | audit2why
```

| 选项 | 说明                                                                                         |
|---------|------------------------------------------------------------------------------------------------------|
| `-v`    | 切换到详细模式                                                                             |
| `-w`    | 翻译 SELinux 拒绝的原因并建议解决它的方案（默认选项） |

### 进一步了解 SELinux

`audit2allow` 命令从 "audit" 文件中的一行创建一个模块，以允许 SELinux 操作（当没有模块存在时）：

```bash
audit2allow [-mM]
```

示例：

```bash
sudo cat /var/log/audit/audit.log | grep AVC | grep denied | tail -1 | audit2allow -M mylocalmodule
```

| 选项 | 说明                                       |
|---------|----------------------------------------------------|
| `-m`    | 仅创建模块（`*.te`）                    |
| `-M`    | 创建模块、编译并打包它（`*.pp`） |

#### 配置示例

执行命令后，系统返回命令提示符但未显示预期的结果：屏幕上没有错误消息。

- **步骤 1**：读取日志文件，知道我们感兴趣的消息是 AVC 类型（SELinux），被拒绝（denied）且是最新的（因此是最后一条）。

```bash
sudo cat /var/log/audit/audit.log | grep AVC | grep denied | tail -1
```

消息被正确隔离，但对我们没有帮助。

- **步骤 2**：使用 `audit2why` 命令读取隔离的消息，以获取可能包含我们问题解决方案的更明确的消息（通常是一个需要设置的布尔值）。

```bash
sudo cat /var/log/audit/audit.log | grep AVC | grep denied | tail -1 | audit2why
```

有两种情况：要么我们可以放置上下文或填写布尔值，要么我们必须转到步骤 3 创建我们自己的上下文。

- **步骤 3**：创建您自己的模块。

```bash
$ sudo cat /var/log/audit/audit.log | grep AVC | grep denied | tail -1 | audit2allow -M mylocalmodule
Generating type enforcement: mylocalmodule.te
Compiling policy: checkmodule -M -m -o mylocalmodule.mod mylocalmodule.te
Building package: semodule_package -o mylocalmodule.pp -m mylocalmodule.mod

$ sudo semodule -i mylocalmodule.pp
```
