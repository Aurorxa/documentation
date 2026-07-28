---
title: Systemd 单元加固
author: Julian Patocki
contributors: Steven Spencer, Ganna Zhyrnova
tags:
    - security
    - systemd
    - capabilities
---

## 前提条件

* 熟悉命令行工具
* 对 `systemd` 和文件权限有基本了解
* 能够阅读手册页

## 简介

许多服务以超出其正常运行所需的权限运行。`systemd` 提供了许多工具，通过强制执行安全措施和限制权限，帮助在进程被攻陷时将风险降到最低。

## 目标

* 提高 `systemd` 单元的安全性

## 免责声明

本指南解释了加固 `systemd` 单元的机制，不涵盖任何特定单元的正确配置。一些概念被过度简化。理解它们和使用的某些命令需要更深入地探索该主题。

## 资源

* [`SYSTEMD.EXEC(5)` 手册页](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html)
* [`Capabilities(7)` 手册页](https://man7.org/linux/man-pages/man7/capabilities.7.html)

## 分析

`systemd` 包含一个很好的工具，可对 `systemd` 单元的整体安全配置提供快速概览。
`systemd-analyze security` 提供 `systemd` 单元安全配置的快速概览。以下是一个全新安装的 `httpd` 的得分：

```bash
[user@rocky-vm ~]$ systemd-analyze security httpd
  NAME                                  DESCRIPTION                                                              EXPOSURE
✗ RootDirectory=/RootImage=             Service runs within the host's root directory                            0.1
  SupplementaryGroups=                  Service runs as root, option does not matter
  RemoveIPC=                            Service runs as root, option does not apply
✗ User=/DynamicUser=                    Service runs as root user                                                0.4
✗ CapabilityBoundingSet=~CAP_SYS_TIME   Service processes may change the system clock                            0.2
✗ NoNewPrivileges=                      Service processes may acquire new privileges                             0.2
...
...
...
✓ NotifyAccess=                         Service child processes cannot alter service state
✓ PrivateMounts=                        Service cannot install system mounts
✗ UMask=                                Files created by service are world-readable by default                   0.1

→ Overall exposure level for httpd.service: 9.2 UNSAFE 😨
```

## Capabilities（能力）

能力（capabilities）的概念可能会让人非常困惑。理解它对于提高 `systemd` 单元的安全性至关重要。以下是 `Capabilities(7)` 手册页的摘录：

```text
For the purpose of performing permission checks, traditional UNIX implementations distinguish two categories of processes: privileged processes (whose effective user ID is 0, referred to as superuser or root), and unprivileged processes (whose effective UID is nonzero).  Privileged processes bypass all kernel permission checks, while  unprivileged  processes are subject to full permission checking based on the process's credentials (usually: effective UID, effective GID, and supplementary group list).

Starting  with  Linux 2.2, Linux divides the privileges traditionally associated with superuser into distinct units, known as capabilities, which can be independently enabled and disabled. Capabilities are a per-thread attribute.
```

这基本上意味着能力可以授予非特权进程一些 `root` 权限，但也可以限制由 `root` 运行的进程的权限。

目前有 41 种能力。这意味着 `root` 用户的权限有 41 组权限。以下是一些示例：

* **CAP_CHOWN**：对文件 UID 和 GID 进行任意更改
* **CAP_KILL**：绕过发送信号的权限检查
* **CAP_NET_BIND_SERVICE**：将套接字绑定到互联网域特权端口（端口号小于 1024）

`Capabilities(7)` 手册页包含完整列表。

有两种类型的能力：

* 文件能力（File capabilities）
* 线程能力（Thread capabilities）

## 文件能力

文件能力允许将权限与可执行文件关联，类似于 `suid`。它们包括存储在扩展属性中的三个集合：`Permitted`、`Inheritable` 和 `Effective`。

完整解释请参考 `Capabilities(7)` 手册页。

文件能力不会影响单元的整体暴露水平，因此与本指南仅略微相关。不过理解它们可能会有帮助。因此，快速演示一下：

让我们尝试以非特权用户身份在默认（特权）端口 80 上运行 `httpd`：

```bash
[user@rocky-vm ~]$ sudo -u apache /usr/sbin/httpd
(13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:80
no listening sockets available, shutting down
```

正如预期，操作失败。让我们为 `httpd` 二进制文件配备前面提到的 **CAP_NET_BIND_SERVICE** 和 **CAP_DAC_OVERRIDE**（为了本练习的目的，覆盖日志和 pid 文件的文件权限检查）并重试：

```bash
[user@rocky-vm ~]$ sudo setcap "cap_net_bind_service=+ep cap_dac_override=+ep" /usr/sbin/httpd
[user@rocky-vm ~]$ sudo -u apache /usr/sbin/httpd
[user@rocky-vm ~]$ curl --head localhost
HTTP/1.1 403 Forbidden
...
```

正如预期，Web 服务器可以成功启动。

## 线程能力

线程能力适用于进程及其子进程。有五种线程能力集：

* Permitted
* Inheritable
* Effective
* Bounding
* Ambient

完整解释请参考 `Capabilities(7)` 手册页。

您已经确定 `httpd` 不需要 `root` 用户所有可用的权限。让我们从 `httpd` 二进制文件中移除之前授予的能力，启动 `httpd` 守护进程，并检查其权限：

```bash
[user@rocky-vm ~]$ sudo setcap -r /usr/sbin/httpd
[user@rocky-vm ~]$ sudo systemctl start httpd
[user@rocky-vm ~]$ grep Cap /proc/$(pgrep --uid 0 httpd)/status
CapInh: 0000000000000000
CapPrm: 000001ffffffffff
CapEff: 000001ffffffffff
CapBnd: 000001ffffffffff
CapAmb: 0000000000000000
[user@rocky-vm ~]$ capsh --decode=000001ffffffffff
0x000001ffffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read,cap_perfmon,cap_bpf,cap_checkpoint_restore
```

主 `httpd` 进程以所有可用的能力运行，即使大多数是不需要的。

## 限制能力

`systemd` 将能力集缩减为以下内容：

* **CapabilityBoundingSet**：限制在 `execve` 期间获得的能力
* **AmbientCapabilities**：如果您想以非特权用户身份执行进程但仍然想给予它一些能力，这是有用的

为了在软件包更新后保留配置，在 `/lib/systemd/system/httpd.service.d/` 目录中创建一个 `override.conf` 文件。

知道该服务需要访问一个特权端口并且它以 `root` 身份启动，但将其线程作为 `apache` fork，因此需要在 `/lib/systemd/system/httpd.service.d/override.conf` 文件的 `[Service]` 部分中指定以下能力：

```bash
[Service]
CapabilityBoundingSet=CAP_NET_BIND_SERVICE CAP_SETUID CAP_SETGID
```

可以将整体暴露级别从 `UNSAFE` 降低到 `MEDIUM`。

```bash
[user@rocky-vm ~]$ sudo systemctl daemon-reload
[user@rocky-vm ~]$ sudo systemctl restart httpd
[user@rocky-vm ~]$ systemd-analyze security --no-pager httpd | grep Overall
→ Overall exposure level for httpd.service: 7.1 MEDIUM 😐
```

然而，此进程仍以 `root` 身份运行。可以通过以 `apache` 身份独家运行它来进一步降低暴露级别。

除了访问端口 80 之外，该进程还需要写入位于 `/etc/httpd/logs/` 的日志，并能够创建 `/run/httpd/` 并写入其中。前者通过使用 `chown` 更改权限，后者通过使用 `systemd-tmpfiles` 实用工具来实现。您可以将其与选项 `--create` 一起使用，以在不重启的情况下创建文件，但从现在开始，它将在每次系统启动时自动创建。

```bash
[user@rocky-vm ~]$ sudo chown -R apache:apache /etc/httpd/logs/
[user@rocky-vm ~]$ echo 'd /run/httpd 0755 apache apache -' | sudo tee /etc/tmpfiles.d/httpd.conf
d /run/httpd 0755 apache apache -
[user@rocky-vm ~]$ sudo systemd-tmpfiles --create /etc/tmpfiles.d/httpd.conf
[user@rocky-vm ~]$ ls -ld /run/httpd/
drwxr-xr-x. 2 apache apache 40 Jun 30 08:29 /run/httpd/
```

您需要调整 `/lib/systemd/system/httpd.service.d/override.conf` 中的配置。您需要使用 **AmbientCapabilities** 授予新能力。如果 `httpd` 在启动时启用，则必须在 `[Unit]` 部分中扩展依赖项，以使服务在临时文件创建后启动。

```bash
[Unit]
After=systemd-tmpfiles-setup.service

[Service]
User=apache
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
```

```bash
[user@rocky-vm ~]$ sudo systemctl daemon-reload
[user@rocky-vm ~]$ sudo systemctl restart httpd
[user@rocky-vm ~]$ grep Cap /proc/$(pgrep httpd | head -1)/status
CapInh: 0000000000000400
CapPrm: 0000000000000400
CapEff: 0000000000000400
CapBnd: 0000000000000400
CapAmb: 0000000000000400
[user@rocky-vm ~]$ capsh --decode=0000000000000400
0x0000000000000400=cap_net_bind_service
[user@rocky-vm ~]$ systemd-analyze security --no-pager httpd | grep Overall
→ Overall exposure level for httpd.service: 6.5 MEDIUM 😐
```

## 文件系统限制

控制进程创建的文件的权限是通过设置 `UMask` 来完成的。
`UMask` 参数通过执行按位操作修改默认文件权限。这主要将默认权限设置为八进制 `0644`（`-rw-r--r--`），默认 `UMask` 是 `0022`。这意味着 `UMask` 不会更改默认设置：

```bash
[user@rocky-vm ~]$ printf "%o\n" $(echo $(( 00644 &  ~00022 )))
644
```

假设守护进程创建的文件的期望权限集是 `0640`（`-rw-r-----`），您可以将 `UMask` 设置为 `7137`。即使默认权限设置为 `7777`，它也能实现目标：

```bash
[user@rocky-vm ~]$ printf "%o\n" $(echo $(( 07777 &  ~07137  )))
640
```

此外：

* `ProtectSystem=`：*"如果设置为 '`strict`'，整个文件系统层次结构以只读方式挂载，除了 API 文件系统子树 `/dev/`、`/proc/` 和 `/sys/`（使用 `PrivateDevices=`、`ProtectKernelTunables=`、`ProtectControlGroups=` 保护这些目录）。"*
* `ReadWritePaths=`：使特定路径再次可写
* `ProtectHome=`：使 `/home/`、`/root` 和 `/run/user` 不可访问
* `PrivateDevices=`：关闭对物理设备的访问，仅允许访问伪设备如 `/dev/null`、`/dev/zero`、`/dev/random`
* `ProtectKernelTunables=`：使 `/proc/` 和 `/sys/` 只读
* `ProtectControlGroups=`：使 `cgroups` 可只读访问
* `ProtectKernelModules=`：拒绝显式模块加载
* `ProtectKernelLogs=`：限制对内核日志缓冲区的访问
* `ProtectProc=`：*"当设置为 'invisible' 时，属于其他用户的进程在 /proc/ 中隐藏。"*
* `ProcSubset=`：*"如果为 'pid'，所有与进程管理和内省不直接关联的文件和目录在 `/proc/` 文件系统中对单元进程不可见。"*

也可以限制可执行路径。守护进程只需要执行其二进制文件和库。`ldd` 实用工具可以告诉我们二进制文件使用了哪些库：

```bash
[user@rocky-vm ~]$ ldd /usr/sbin/httpd
        linux-vdso.so.1 (0x00007ffc0e823000)
        libpcre.so.1 => /lib64/libpcre.so.1 (0x00007fa360d61000)
        libselinux.so.1 => /lib64/libselinux.so.1 (0x00007fa360d34000)
        libaprutil-1.so.0 => /lib64/libaprutil-1.so.0 (0x00007fa360d05000)
        libcrypt.so.2 => /lib64/libcrypt.so.2 (0x00007fa360ccb000)
        libexpat.so.1 => /lib64/libexpat.so.1 (0x00007fa360c9a000)
        libapr-1.so.0 => /lib64/libapr-1.so.0 (0x00007fa360c5a000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fa360a00000)
        libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007fa360964000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fa360e70000)
        libuuid.so.1 => /lib64/libuuid.so.1 (0x00007fa360c4e000)
        libm.so.6 => /lib64/libm.so.6 (0x00007fa360889000)
```

以下行将追加到 `override.conf` 文件的 `[Service]` 部分：

```bash
UMask=7177
ProtectSystem=strict
ReadWritePaths=/run/httpd /etc/httpd/logs

ProtectHome=true
PrivateDevices=true
ProtectKernelTunables=true
ProtectControlGroups=true
ProtectKernelModules=true
ProtectKernelLogs=true
ProtectProc=invisible
ProcSubset=pid

NoExecPaths=/
ExecPaths=/usr/sbin/httpd /lib64
```

让我们重新加载配置并检查对得分的影响：

```bash
[user@rocky-vm ~]$ sudo systemctl daemon-reload
[user@rocky-vm ~]$ sudo systemctl restart httpd
[user@rocky-vm ~]$ systemd-analyze security --no-pager httpd | grep Overall
→ Overall exposure level for httpd.service: 4.9 OK 🙂
```

## 系统限制

各种参数可以限制系统操作以增强安全性：

* `NoNewPrivileges=`：确保进程不能通过 `setuid`、`setgid` 位和文件系统能力获得新权限
* `ProtectClock=`：拒绝写入系统时钟和硬件时钟
* `SystemCallArchitectures=`：如果设置为 `native`，进程只能进行本机 `syscalls`（在大多数情况下是 `x86-64`）
* `RestrictNamespaces=`：命名空间主要与容器相关，因此可以为此单元限制
* `RestrictSUIDSGID=`：阻止进程在文件上设置 `setuid` 和 `setgid` 位
* `LockPersonality=`：防止执行域被更改，这可能仅对运行遗留应用程序或为其他类 Unix 系统设计的软件有用
* `RestrictRealtime=`：实时调度仅对需要严格时间保证的应用程序相关，例如工业控制系统、音频/视频处理和科学模拟
* `RestrictAddressFamilies=`：限制可用的套接字地址族；可以设置为 `AF_(INET|INET6)` 以仅允许 IPv4 和 IPv6 套接字；某些服务需要 `AF_UNIX` 用于内部通信和日志记录
* `MemoryDenyWriteExecute=`：确保进程不能分配既可写又可执行的新内存区域，防止某些类型的攻击，其中恶意代码被注入可写内存然后执行；可能导致 JavaScript、Java 或 .NET 使用的 JIT 编译器失败
* `ProtectHostname=`：防止进程使用 `sethostname()`、`setdomainname()` 系统调用

让我们将以下内容追加到 `override.conf` 文件，重新加载配置并检查对得分的影响：

```bash
NoNewPrivileges=true
ProtectClock=true
SystemCallArchitectures=native
RestrictNamespaces=true
RestrictSUIDSGID=true
LockPersonality=true
RestrictRealtime=true
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
MemoryDenyWriteExecute=true
ProtectHostname=true
```

```bash
[user@rocky-vm ~]$ sudo systemctl daemon-reload
[user@rocky-vm ~]$ sudo systemctl restart httpd
[user@rocky-vm ~]$ systemd-analyze security --no-pager httpd | grep Overall
→ Overall exposure level for httpd.service: 3.0 OK 🙂
```

## 系统调用过滤

限制系统调用可能不容易。很难确定某些守护进程需要调用哪些系统调用才能正常工作。

`strace` 实用工具可以帮助确定创建了哪些系统调用。选项 `-f` 指定跟随 fork 的进程，`-o` 将输出保存到名为 `httpd.strace` 的文件。

```bash
[user@rocky-vm ~]$ sudo strace -f -o httpd.strace /usr/sbin/httpd
```

在运行该进程一段时间并与它交互后，停止执行以检查输出：

```bash
[user@rocky-vm ~]$ awk '{print $2}' httpd.strace | cut -d '(' -f 1 | sort | uniq | sed '/^[^a-zA-Z0-9]*$/d' | wc -l
79
```

该程序在其运行期间创建了 79 个唯一的系统调用。
您可以使用以下一行命令设置允许的系统调用列表：

```bash
[user@rocky-vm ~]$ echo SystemCallFilter=$(awk '{print $2}' httpd.strace | cut -d '(' -f 1 | sort | uniq | sed '/^[^a-zA-Z0-9]*$/d' | tr "\n" " ") | sudo tee -a /lib/systemd/system/httpd.service.d/override.conf
...
...
...
[user@rocky-vm ~]$ sudo systemctl daemon-reload
[user@rocky-vm ~]$ sudo systemctl restart httpd
[user@rocky-vm ~]$ systemd-analyze security --no-pager httpd | grep Overall
→ Overall exposure level for httpd.service: 1.5 OK 🙂
[user@rocky-vm ~]$ curl --head localhost
HTTP/1.1 403 Forbidden
```

Web 服务器仍在运行，暴露已显著降低。

上述方法是精确的。如果遗漏了某个系统调用，可能会导致程序崩溃。`systemd` 将系统调用分组为预定义的集合。为了使限制系统调用更容易，可以在允许或禁止列表中设置整个组，而不是设置单个系统调用。要查找这些列表：

```bash
[user@rocky-vm ~]$ systemd-analyze syscall-filter
@default
    # System calls that are always permitted
    arch_prctl
    brk
    cacheflush
    clock_getres
...
...
...
```

组内的系统调用可能重叠，特别是某些组包含其他组。因此，可以通过使用 `~` 符号指定单个调用或组被禁止。`override.conf` 文件中的以下指令应对此单元有效：

```bash
SystemCallFilter=@system-service
SystemCallFilter=~@privileged @resources @mount @swap @reboot
```

## 结论

大多数 `systemd` 单元的默认安全配置是宽松的。加固它们可能需要一些时间，但这是值得的，尤其是在暴露于互联网的较大环境中。如果攻击者利用漏洞或错误配置，加固的单元可能会阻止他们控制系统。
