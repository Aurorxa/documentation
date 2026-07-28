---
title: Grep 命令
author: tianci li
contributors: 
tags:
  - grep
---

# `grep` 命令

`grep` 命令用于过滤单个或多个文件的内容。此命令工具存在一些变体，例如 `egrep (grep -E)` 和 `fgrep (grep -f)`。更多信息请参阅 [grep 手册页](https://www.gnu.org/software/grep/manual/)。

`grep` 命令的用法为：

```text
grep [OPTIONS] PATTERN [FILE...]
grep [OPTIONS] -e PATTERN ... [FILE...]
grep [OPTIONS] -f FILE ... [FILE...]
```

选项主要分为四个部分：

* 匹配控制
* 输出控制
* 内容行控制
* 目录或文件控制

匹配控制：

| 选项                   | 说明                                                                 |
|:----------------------:|----------------------------------------------------------------------|
| -E (--extended-regexp) | 启用 ERE（扩展正则表达式）                                             |
| -P (--perl-regexp)     | 启用 PCRE（Perl 兼容正则表达式）                                       |
| -G (--basic-regexp)    | 启用 BRE（基本正则表达式），默认选项                                     |
| -e (--regexp=PATTERN)  | 模式匹配，可指定多个 -e 选项                                            |
| -i                     | 忽略大小写                                                             |
| -w                     | 精确匹配整个单词                                                       |
| -f FILE                | 从 FILE 中获取模式，每行一个                                          |
| -x                     | 模式匹配整行                                                           |
| -v                     | 选择不匹配的内容行                                                      |

输出控制：

| 选项                     | 说明                                               |
|:------------------------:|:---------------------------------------------------|
| -m NUM                   | 输出前几个匹配结果                                    |
| -n                       | 在输出中打印行号                                      |
| -H                       | 匹配多个文件的内容时，在行首显示文件名。这是默认设置       |
| -h                       | 匹配多个文件的内容时，不在行首显示文件名                 |
| -o                       | 仅打印匹配内容，不输出整行                              |
| -q                       | 不输出正常信息                                        |
| -s                       | 不输出错误消息                                        |
| -r                       | 对目录进行递归匹配                                    |
| -c                       | 根据用户内容打印每个文件匹配到的行数                       |

内容行控制：

| 选项                     | 说明                                               |
|:------------------------:|:---------------------------------------------------|
| -B NUM                   | 打印匹配行前面的 NUM 行上下文                          |
| -A NUM                   | 打印匹配行后面的 NUM 行上下文                          |
| -C NUM                   | 打印 NUM 行输出上下文                                |

目录或文件控制：

| 选项                     | 说明                                                                                               |
|:------------------------:|----------------------------------------------------------------------------------------------------|
| --include=FILE_PATTERN   | 仅搜索匹配 FILE_PATTERN 的文件。文件名通配符支持 *, ?, [], [^], [-], {..}, {,}                        |
| --exclude=FILE_PATTERN   | 跳过匹配 FILE_PATTERN 的文件。文件名通配符支持 *, ?, [], [^], [-], {..}, {,}                          |
| --exclude-dir=PATTERN    | 排除指定的目录名。目录名支持 *, ?, [], [^], [-], {..}, {,}                                          |
| --exclude-from=FILE      | 从文件内容中排除指定的文件                                                                             |

## 使用示例

1. -f 选项和 -o 选项

    ```bash
    Shell > cat /root/a
    abcdef
    123456
    338922549
    24680
    hello world

    Shell > cat /root/b
    12345
    test
    world
    aaaaa

    # 将文件 b 的每一行作为匹配模式，输出与文件 a 匹配的行
    Shell > grep -f /root/b /root/a
    123456
    hello world

    Shell > grep -f /root/b /root/a -o
    12345
    world
    ```

2. 多模式匹配（使用 -e 选项）

    ```bash
    Shell > echo -e "a\nab\nbc\nbcde" | grep -e 'a' -e 'cd'
    a
    ab
    bcde
    ```

    或：

    ```bash
    Shell > echo -e "a\nab\nbc\nbcde" | grep -E "a|cd"
    a
    ab
    bcde
    ```

3. 从配置文件中删除空行和注释行

    ```bash
    Shell > grep -v -E "^$|^#" /etc/chrony.conf
    server ntp1.tencent.com iburst
    server ntp2.tencent.com iburst
    server ntp3.tencent.com iburst
    server ntp4.tencent.com iburst
    driftfile /var/lib/chrony/drift
    makestep 1.0 3
    rtcsync
    keyfile /etc/chrony.keys
    leapsectz right/UTC
    logdir /var/log/chrony
    ```

4. 打印匹配的前 5 个结果

    ```bash
    Shell > seq 1 20 | grep -m 5 -E "[0-9]{2}"
    10
    11
    12
    13
    14
    ```

    或：

    ```bash
    Shell > seq 1 20 | grep -m 5  "[0-9]\{2\}"
    10
    11
    12
    13
    14
    ```

5. -B 选项和 -A 选项

    ```bash
    Shell > seq 1 20 | grep -B 2 -A 3 -m 5 -E "[0-9]{2}"
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    ```

6. -C 选项

    ```bash
    Shell > seq 1 20 | grep -C 3 -m 5 -E "[0-9]{2}"
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    ```

7. -c 选项

    ```bash
    Shell > cat /etc/ssh/sshd_config | grep  -n -i -E "port"
    13:# If you want to change the port on a SELinux system, you have to tell
    15:# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
    17:#Port 22
    99:# WARNING: 'UsePAM no' is not supported in RHEL and may cause several
    105:#GatewayPorts no

    Shell > cat /etc/ssh/sshd_config | grep -E -i "port" -c
    5
    ```

8. -v 选项

    ```bash
    Shell > cat /etc/ssh/sshd_config | grep -i -v -E "port" -c
    140
    ```

9. 过滤目录中包含匹配字符串行内容的文件（排除子目录中的文件）

    ```bash
    Shell > grep -i -E "port" /etc/n*.conf -n
    /etc/named.conf:11:     listen-on port 53 { 127.0.0.1; };
    /etc/named.conf:12:     listen-on-v6 port 53 { ::1; };
    /etc/nsswitch.conf:32:# winbind                 Use Samba winbind support
    /etc/nsswitch.conf:33:# wins                    Use Samba wins support
    ```
  
10. 过滤目录中包含匹配字符串行内容的文件（包括或排除子目录中的文件或目录）

    包含多个文件的语法：

    ```bash
    Shell > grep -n -i -r -E  "port" /etc/ --include={0..20}_*
    /etc/grub.d/20_ppc_terminfo:26:export TEXTDOMAIN=grub
    /etc/grub.d/20_ppc_terminfo:27:export TEXTDOMAINDIR=/usr/share/locale
    /etc/grub.d/20_linux_xen:26:export TEXTDOMAIN=grub
    /etc/grub.d/20_linux_xen:27:export TEXTDOMAINDIR="${datarootdir}/locale"
    /etc/grub.d/20_linux_xen:46:# Default to disabling partition uuid support to maintian compatibility with
    /etc/grub.d/10_linux:26:export TEXTDOMAIN=grub
    /etc/grub.d/10_linux:27:export TEXTDOMAINDIR="${datarootdir}/locale"
    /etc/grub.d/10_linux:47:# Default to disabling partition uuid support to maintian compatibility with

    Shell > grep -n -i -r -E  "port" /etc/ --include={{0..20}_*,sshd_config} -c
    /etc/ssh/sshd_config:5
    /etc/grub.d/20_ppc_terminfo:2
    /etc/grub.d/10_reset_boot_success:0
    /etc/grub.d/12_menu_auto_hide:0
    /etc/grub.d/20_linux_xen:3
    /etc/grub.d/10_linux:3
    ```

    如果需要排除单个目录，使用以下语法：

    ```bash
    Shell > grep -n -i -r -E  "port" /etc/ --exclude-dir=selin[u]x
    ```

    如果需要排除多个目录，使用以下语法：

    ```bash
    Shell > grep -n -i -r -E  "port" /etc/ --exclude-dir={selin[u]x,"profile.d",{a..z}ki,au[a-z]it}
    ```

    如果需要排除单个文件，使用以下语法：

    ```bash
    Shell > grep -n -i -r -E  "port" /etc/ --exclude=sshd_config
    ```

    如果需要排除多个文件，使用以下语法：

    ```bash
    Shell > grep -n -i -r -E  "port" /etc/ --exclude={ssh[a-z]_config,*.conf,services}
    ```

    如果需要同时排除多个文件和目录，使用以下语法：

    ```bash
    Shell > grep -n -i -r -E  "port" /etc/ --exclude-dir={selin[u]x,"profile.d",{a..z}ki,au[a-z]it} --exclude={ssh[a-z]_config,*.conf,services,[0-9][0-9]*}
    ```

11. 统计当前机器的所有 IPv4 地址

    ```bash
    Shell > ip a | grep -o  -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | grep -v -E "127|255"
    192.168.100.3
    ```
