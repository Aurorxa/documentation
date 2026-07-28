---
title: Awk 命令
author: tianci li
contributors:
tags:
  - awk
---

# `awk` 命令

1977 年，一个名为 'awk' 的编程语言级文本处理工具诞生于贝尔实验室。其名称来源于三位著名人物姓氏的首字母：

* Alfred **A**ho
* Peter **W**einberger
* Brian **K**ernighan

与 shell（bash、csh、zsh 和 ksh）类似，`awk` 也有随着历史发展而来的衍生版本：

* `awk`：1977 年诞生于贝尔实验室。
* `nawk`（new awk）：1985 年创建，是 `awk` 的更新和增强版本。随 Unix System V Release 3.1（1987）广泛使用。`oawk` 指旧版本的 `awk`。
* `gawk`（GNU awk）：由 Paul Rubin 于 1986 年编写。GNU 项目于 1984 年诞生。
* `mawk`：由 Mike Brennan 于 1996 年编写，是 `awk` 编程语言的解释器。
* `jawk`：`awk` 的 JAVA 实现

在 GNU/Linux 操作系统中，通常 `awk` 指的是 `gawk`。不过，某些发行版（如 Ubuntu 或 Debian）使用 `mawk` 作为其默认 `awk`。

在所有近期版本的 Rocky Linux 中，`awk` 指的是 `gawk`。

```bash
Shell > whereis awk
awk: /usr/bin/awk /usr/libexec/awk /usr/share/awk /usr/share/man/man1/awk.1.gz

Shell > ls -l /usr/bin/awk
lrwxrwxrwx. 1 root root 4 4月  16 2022 /usr/bin/awk -> gawk

Shell > rpm -qf /usr/bin/awk
gawk-4.2.1-4.el8.x86_64
```

更多信息请参考 [gawk 手册](https://www.gnu.org/software/gawk/manual/ "gawk manual")。

虽然 `awk` 是一个用于处理文本的工具，但它具有一些编程语言的特性：

* 变量
* 流程控制（循环）
* 数据类型
* 逻辑运算
* 函数
* 数组
* ...

**`awk` 的工作原理**：类似于关系数据库，支持对字段（列）和记录（行）进行处理。默认情况下，`awk` 将文件的每一行视为一条记录，并将这些记录放入内存中逐行处理，每行的每个部分被视为记录中的一个字段。默认情况下，分隔不同字段的分隔符使用空格和制表符（Tab），数字代表行记录中不同的字段。引用多个字段时，用逗号或制表符分隔。

一个易于理解的简单示例：

```bash
Shell > df -hT
| 1             |     2        |  3    |  4   |  5    |   6   |   7            | 8       |
|Filesystem     |    Type      | Size  | Used | Avail | Use%  | Mounted        | on      |←← 1（第一行）
|devtmpfs       |    devtmpfs  | 1.8G  |   0  | 1.8G  |  0%   | /dev           |         |←← 2
|tmpfs          |    tmpfs     | 1.8G  |    0 | 1.8G  |  0%   | /dev/shm       |         |←← 3
|tmpfs          |    tmpfs     | 1.8G  | 8.9M | 1.8G  |  1%   | /run           |         |←← 4
|tmpfs          |    tmpfs     | 1.8G  |   0  | 1.8G  |  0%   | /sys/fs/cgroup |         |←← 5
|/dev/nvme0n1p2 |    ext4      | 47G   | 2.6G |  42G  |  6%   | /              |         |←← 6
|/dev/nvme0n1p1 |    xfs       | 1014M | 182M | 833M  |  18%  | /boot          |         |←← 7
|tmpfs          |    tmpfs     | 364M  |   0  | 364M  |  0%   | /run/user/0    |         |←← 8（末尾行）

Shell > df -hT | awk '{print $1,$2}'
Filesystem  Type
devtmpfs devtmpfs
tmpfs tmpfs
tmpfs tmpfs
tmpfs tmpfs
/dev/nvme0n1p2 ext4
/dev/nvme0n1p1 xfs
tmpfs tmpfs

# $0：引用整个文本内容。
Shell > df -hT | awk '{print $0}'
Filesystem     Type      Size   Used  Avail Use% Mounted on
devtmpfs       devtmpfs  1.8G     0  1.8G    0%  /dev
tmpfs          tmpfs     1.8G     0  1.8G    0%  /dev/shm
tmpfs          tmpfs     1.8G  8.9M  1.8G    1%  /run
tmpfs          tmpfs     1.8G     0  1.8G    0%  /sys/fs/cgroup
/dev/nvme0n1p2 ext4       47G  2.6G   42G    6%  /
/dev/nvme0n1p1 xfs      1014M  182M  833M   18%  /boot
tmpfs          tmpfs     364M     0  364M    0%  /run/user/0
```

## `awk` 的使用说明

`awk` 的用法为 - `awk option  'pattern {action}'  FileName`

**pattern**：在文本中查找特定内容
**action**：动作指令
**{ }**：根据特定模式将一些指令分组

| 选项                                 | 说明                                                                              |
|--------------------------------------|---------------------------------------------------------------------------------|
| -f program-file  --file program-file | 从文件中读取 `awk` 程序源文件                                                        |
| -F FS                                | 指定用于分隔字段的分隔符。这里的 'FS' 是 `awk` 中的内置变量，默认值为空格或制表符         |
| -v var=value                         | 变量赋值                                                                          |
| --posix                              | 开启兼容模式                                                                       |
| --dump-variables=[file]              | 将 `awk` 中的全局变量写入文件。如果未指定文件，默认文件为 awkvars.out                    |
| --profile=[file]                     | 将性能分析数据写入特定文件。如果未指定文件，默认文件为 awkprof.out                       |

| pattern                | 说明                       |
| :---                   | :---                       |
| BEGIN{ }               | 在读取所有行记录之前执行的动作     |
| END{ }                 | 在读取所有行记录之后执行的动作     |
| /regular  expression/  | 对每个输入行记录进行正则表达式匹配   |
| pattern && pattern     | 逻辑与运算                     |
| pattern \|\| pattern   | 逻辑或运算                     |
| !pattern               | 逻辑取反运算                  |
| pattern1,pattern2      | 指定模式范围，匹配该范围内的所有行记录 |

`awk` 功能强大，涉及的知识很多，因此部分内容将在后续讲解。

### `printf` 命令

在正式学习 `awk` 之前，初学者需要了解 `printf` 命令。

`printf`：格式化并打印数据。其用法为 - `printf FORMAT [ARGUMENT]...`

**FORMAT**：用于控制输出内容。支持以下常见转义序列：

* **\a** - 警报（BEL）
* **\b** - 退格
* **\f** - 换页
* **\n** - 换行
* **\r** - 回车
* **\t** - 水平制表符
* **\v** - 垂直制表符
* **%Ns** - 输出字符串。N 代表字符串数量，例如：`%s %s %s`
* **%Ni** - 输出整数。N 代表输出整数的个数，例如：`%i %i`
* **%m\.nf** - 输出浮点数。m 代表输出总位数，n 代表小数点后的位数。例如：`%8.5f`

**ARGUMENT**：如果是文件，需要进行一些预处理才能正确输出。

```bash
Shell > cat /tmp/printf.txt
ID      Name    Age     Class
1       Frank   20      3
2       Jack    25      5
3       Django  16      6
4       Tom     19      7

# 错误语法示例：
Shell > printf '%s %s $s\n' /tmp/printf.txt
/tmp/printf.txt

# 更改文本格式
Shell > printf '%s' $(cat /tmp/printf.txt)
IDNameAgeClass1Frank2032Jack2553Django1664Tom197
# 更改文本格式
Shell > printf '%s\t%s\t%s\n' $(cat /tmp/printf.txt)
ID      Name    Age
Class   1       Frank
20      3       2
Jack    25      5
3       Django  16
6       4       Tom
19      7

Shell > printf "%s\t%s\t%s\t%s\n" a b c d 1 2 3 4
a       b       c       d
1       2       3       4
```

RockyLinux OS 中不存在 `print` 命令。你只能在 `awk` 中使用 `print`，它与 `printf` 的区别在于它会在每行末尾自动添加换行符。例如：

```bash
Shell > awk '{printf $1 "\t" $2"\n"}' /tmp/printf.txt
ID      Name
1       Frank
2       Jack
3       Django
4       Tom

Shell > awk '{print $1 "\t" $2}' /tmp/printf.txt
ID      Name
1       Frank
2       Jack
3       Django
4       Tom
```

## 基本用法示例

1. 从文件中读取 `awk` 程序源文件

    ```bash
    Shell > vim /tmp/read-print.awk
    #!/bin/awk
    {print $6}

    Shell > df -hT | awk -f /tmp/read-print.awk
    Use%
    0%
    0%
    1%
    0%
    6%
    18%
    0%
    ```

2. 指定分隔符

    ```bash
    Shell > awk -F ":" '{print $1}' /etc/passwd
    root
    bin
    daemon
    adm
    lp
    sync
    ...

    Shell > tail -n 5 /etc/services | awk -F "\/" '{print $2}'
    awk: warning: escape sequence `\/' treated as plain `/'
    axio-disc       35100
    pmwebapi        44323
    cloudcheck-ping 45514
    cloudcheck      45514
    spremotetablet  46998
    ```

    也可以使用单词作为分隔符。括号表示这是一个整体分隔符，`|` 表示或。

    ```bash
    Shell > tail -n 5 /etc/services | awk -F "(tcp)|(udp)" '{print $1}'
    axio-disc       35100/
    pmwebapi        44323/
    cloudcheck-ping 45514/
    cloudcheck      45514/
    spremotetablet  46998/
    ```

3. 变量赋值

    ```bash
    Shell > tail -n 5 /etc/services | awk -v a=123 'BEGIN{print a}{print $1}'
    123
    axio-disc
    pmwebapi
    cloudcheck-ping
    cloudcheck
    spremotetablet
    ```

    将 bash 中用户定义的变量值赋给 awk 的变量。

    ```bash
    Shell > ab=123
    Shell > echo ${ab}
    123
    Shell > tail -n 5 /etc/services | awk -v a=${ab} 'BEGIN{print a}{print $1}'
    123
    axio-disc
    pmwebapi
    cloudcheck-ping
    cloudcheck
    spremotetablet
    ```

4. 将 awk 的全局变量写入文件

    ```bash
    Shell > seq 1 6 | awk --dump-variables '{print $0}'
    1
    2
    3
    4
    5
    6

    Shell > cat /root/awkvars.out
    ARGC: 1
    ARGIND: 0
    ARGV: array, 1 elements
    BINMODE: 0
    CONVFMT: "%.6g"
    ENVIRON: array, 27 elements
    ERRNO: ""
    FIELDWIDTHS: ""
    FILENAME: "-"
    FNR: 6
    FPAT: "[^[:space:]]+"
    FS: " "
    FUNCTAB: array, 41 elements
    IGNORECASE: 0
    LINT: 0
    NF: 1
    NR: 6
    OFMT: "%.6g"
    OFS: " "
    ORS: "\n"
    PREC: 53
    PROCINFO: array, 20 elements
    RLENGTH: 0
    ROUNDMODE: "N"
    RS: "\n"
    RSTART: 0
    RT: "\n"
    SUBSEP: "\034"
    SYMTAB: array, 28 elements
    TEXTDOMAIN: "messages"
    ```

    稍后将介绍这些变量的含义。如需立即查看，[跳转到变量部分](#VARIABLES)。

5. BEGIN{ } 和 END{ }

    ```bash
    Shell > head -n 5 /etc/passwd | awk 'BEGIN{print "UserName:PasswordIdentification:UID:InitGID"}{print $0}END{print "one\ntwo"}'
    UserName:PasswordIdentification:UID:InitGID
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
    one
    two
    ```

6. --profile 选项

    ```bash
    Shell > df -hT | awk --profile 'BEGIN{print "start line"}{print $0}END{print "end line"}'
    start line
    Filesystem     Type      Size  Used Avail Use% Mounted on
    devtmpfs       devtmpfs  1.8G     0  1.8G   0% /dev
    tmpfs          tmpfs     1.8G     0  1.8G   0% /dev/shm
    tmpfs          tmpfs     1.8G  8.9M  1.8G   1% /run
    tmpfs          tmpfs     1.8G     0  1.8G   0% /sys/fs/cgroup
    /dev/nvme0n1p2 ext4       47G  2.7G   42G   6% /
    /dev/nvme0n1p1 xfs      1014M  181M  834M  18% /boot
    tmpfs          tmpfs     363M     0  363M   0% /run/user/0
    end line

    Shell > cat /root/awkprof.out
        # gawk profile, created Fri Dec  8 15:12:56 2023

        # BEGIN rule(s)

        BEGIN {
     1          print "start line"
        }

        # Rule(s)

     8  {
     8          print $0
        }

        # END rule(s)

        END {
     1          print "end line"
        }
    ```

    修改 awkprof.out 文件。

    ```bash
    Shell > vim /root/awkprof.out
    BEGIN {
        print "start line"
    }

    {
        print $0
    }

    END {
        print "end line"
    }

    Shell > df -hT | awk -f /root/awkprof.out
    start line
    Filesystem     Type      Size  Used Avail Use% Mounted on
    devtmpfs       devtmpfs  1.8G     0  1.8G   0% /dev
    tmpfs          tmpfs     1.8G     0  1.8G   0% /dev/shm
    tmpfs          tmpfs     1.8G  8.9M  1.8G   1% /run
    tmpfs          tmpfs     1.8G     0  1.8G   0% /sys/fs/cgroup
    /dev/nvme0n1p2 ext4       47G  2.7G   42G   6% /
    /dev/nvme0n1p1 xfs      1014M  181M  834M  18% /boot
    tmpfs          tmpfs     363M     0  363M   0% /run/user/0
    end line
    ```

7. 通过正则表达式匹配行（记录）<a id="RE"></a>

    ```bash
    Shell > cat /etc/services | awk '/[^0-9a-zA-Z]1[1-9]{2}\/tcp/ {print $0}'
    sunrpc          111/tcp         portmapper rpcbind      # RPC 4.0 portmapper TCP
    auth            113/tcp         authentication tap ident
    sftp            115/tcp
    uucp-path       117/tcp
    nntp            119/tcp         readnews untp   # USENET News Transfer Protocol
    ntp             123/tcp
    netbios-ns      137/tcp                         # NETBIOS Name Service
    netbios-dgm     138/tcp                         # NETBIOS Datagram Service
    netbios-ssn     139/tcp                         # NETBIOS session service
    ...
    ```

8. 逻辑运算（逻辑与、逻辑或、取反）

    逻辑与：&&
    逻辑或：||
    取反：!

    ```bash
    Shell > cat /etc/services | awk '/[^0-9a-zA-Z]1[1-9]{2}\/tcp/ && /175/ {print $0}'
    vmnet           175/tcp                 # VMNET
    ```

    ```bash
    Shell > cat /etc/services | awk '/[^0-9a-zA-Z]9[1-9]{2}\/tcp/ || /91{2}\/tcp/ {print $0}'
    telnets         992/tcp
    imaps           993/tcp                         # IMAP over SSL
    pop3s           995/tcp                         # POP-3 over SSL
    mtp             1911/tcp                        #
    rndc            953/tcp                         # rndc control sockets (BIND 9)
    xact-backup     911/tcp                 # xact-backup
    apex-mesh       912/tcp                 # APEX relay-relay service
    apex-edge       913/tcp                 # APEX endpoint-relay service
    ftps-data       989/tcp                 # ftp protocol, data, over TLS/SSL
    nas             991/tcp                 # Netnews Administration System
    vsinet          996/tcp                 # vsinet
    maitrd          997/tcp                 #
    busboy          998/tcp                 #
    garcon          999/tcp                 #
    #puprouter      999/tcp                 #
    blockade        2911/tcp                # Blockade
    prnstatus       3911/tcp                # Printer Status Port
    cpdlc           5911/tcp                # Controller Pilot Data Link Communication
    manyone-xml     8911/tcp                # manyone-xml
    sype-transport  9911/tcp                # SYPECom Transport Protocol
    ```

    ```bash
    Shell > cat /etc/services | awk '!/(tcp)|(udp)/ {print $0}'
    discard         9/sctp                  # Discard
    discard         9/dccp                  # Discard SC:DISC
    ftp-data        20/sctp                 # FTP
    ftp             21/sctp                 # FTP
    ssh             22/sctp                 # SSH
    exp1            1021/sctp                # RFC3692-style Experiment 1 (*)                [RFC4727]
    exp1            1021/dccp                # RFC3692-style Experiment 1 (*)                [RFC4727]
    exp2            1022/sctp                # RFC3692-style Experiment 2 (*)                [RFC4727]
    exp2            1022/dccp                # RFC3692-style Experiment 2 (*)                [RFC4727]
    ltp-deepspace   1113/dccp               # Licklider Transmission Protocol
    cisco-ipsla     1167/sctp               # Cisco IP SLAs Control Protocol
    rcip-itu        2225/sctp               # Resource Connection Initiation Protocol
    m2ua            2904/sctp               # M2UA
    m3ua            2905/sctp               # M3UA
    megaco-h248     2944/sctp               # Megaco-H.248 text
    ...
    ```

9. 通过字符串定位连续行并打印

    ```bash
    Shell > cat /etc/services | awk '/^ntp/,/^netbios/ {print $0}'
    ntp             123/tcp
    ntp             123/udp                         # Network Time Protocol
    netbios-ns      137/tcp                         # NETBIOS Name Service
    ```

    !!! info

        范围起始：遇到第一个匹配时停止匹配。
        范围结束：遇到第一个匹配时停止匹配。

## 内置变量 {#VARIABLES}

| 变量名     | 说明                                                                   |
| :---:      | :---                                                                   |
| FS         | 输入字段的分隔符。默认为空格或制表符                                          |
| OFS        | 输出字段的分隔符。默认为空格                                                |
| RS         | 输入行记录的分隔符。默认为换行符（\n）                                         |
| ORS        | 输出行记录的分隔符。默认为换行符（\n）                                         |
| NF         | 统计当前行记录中的字段数量                                                   |
| NR         | 统计行记录数量。每处理一行文本后，该变量的值将 +1                                |
| FNR        | 统计行记录数量。处理第二个文件时，NR 变量继续累加，但 FNR 变量重新计数               |
| ARGC       | 命令行参数的数量                                                            |
| ARGV       | 命令行参数数组，下标从 0 开始，ARGV[0] 代表 `awk`                              |
| ARGIND     | 当前正在处理的文件的索引值。第一个文件为 1，第二个文件为 2，依此类推                |
| ENVIRON    | 当前系统的环境变量                                                           |
| FILENAME   | 输出当前正在处理的文件名                                                       |
| IGNORECASE | 忽略大小写                                                                 |
| SUBSEP     | 数组中下标的隔符，默认为 "\034"                                               |

1. FS 和 OFS

    ```bash
    Shell > cat /etc/passwd | awk 'BEGIN{FS=":"}{print $1}'
    root
    bin
    daemon
    adm
    lp
    sync
    ```

    也可以使用 -v 选项为变量赋值。

    ```bash
    Shell > cat /etc/passwd | awk -v FS=":" '{print $1}'
    root
    bin
    daemon
    adm
    lp
    sync
    ```

    使用逗号引用多个字段时，默认的输出分隔符是空格。不过，你可以单独指定输出分隔符。

    ```bash
    Shell > cat /etc/passwd | awk 'BEGIN{FS=":"}{print $1,$2}'
    root x
    bin x
    daemon x
    adm x
    lp x
    ```

    ```bash
    Shell > cat /etc/passwd | awk 'BEGIN{FS=":";OFS="\t"}{print $1,$2}'
    # 或
    Shell > cat /etc/passwd | awk -v FS=":" -v OFS="\t" '{print $1,$2}'
    root    x
    bin     x
    daemon  x
    adm     x
    lp      x
    ```

2. RS 和 ORS

    默认情况下，`awk` 使用换行符来区分每一行记录。

    ```bash
    Shell > echo -e "https://example.com/books/index.html\ntitle//tcp"
    https://example.com/books/index.html
    title//tcp

    Shell > echo -e "https://example.com/books/index.html\ntitle//tcp" | awk 'BEGIN{RS="\/\/";ORS="%%"}{print $0}'
    awk: cmd. line:1: warning: escape sequence `\/' treated as plain `/'
    https:%%example.com/books/index.html
    title%%tcp
    %%             ← 为什么？因为 "print"
    ```

3. NF

    统计当前文本中每行的字段数量。

    ```bash
    Shell > head -n 5 /etc/passwd | awk -F ":" 'BEGIN{RS="\n";ORS="\n"} {print NF}'
    7
    7
    7
    7
    7
    ```

    打印第五个字段。

    ```bash
    Shell > head -n 5 /etc/passwd | awk -F ":" 'BEGIN{RS="\n";ORS="\n"} {print $(NF-2)}'
    root
    bin
    daemon
    adm
    lp
    ```

    打印最后一个字段。

    ```bash
    Shell > head -n 5 /etc/passwd | awk -F ":" 'BEGIN{RS="\n";ORS="\n"} {print $NF}'
    /bin/bash
    /sbin/nologin
    /sbin/nologin
    /sbin/nologin
    /sbin/nologin
    ```

    排除最后两个字段。

    ```bash
    Shell > head -n 5 /etc/passwd | awk -F ":" 'BEGIN{RS="\n";ORS="\n"} {$NF=" ";$(NF-1)=" ";print $0}'
    root x 0 0 root
    bin x 1 1 bin
    daemon x 2 2 daemon
    adm x 3 4 adm
    lp x 4 7 lp
    ```

    排除第一个字段。

    ```bash
    Shell > head -n 5 /etc/passwd | awk -F ":" 'BEGIN{RS="\n";ORS="\n"} {$1=" ";print $0}' | sed -r 's/(^  )//g'
    x 0 0 root /root /bin/bash
    x 1 1 bin /bin /sbin/nologin
    x 2 2 daemon /sbin /sbin/nologin
    x 3 4 adm /var/adm /sbin/nologin
    x 4 7 lp /var/spool/lpd /sbin/nologin
    ```

4. NR 和 FNR

    ```bash
    Shell > tail -n 5 /etc/services | awk '{print NR,$0}'
    1 axio-disc       35100/udp               # Axiomatic discovery protocol
    2 pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    3 cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    4 cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    5 spremotetablet  46998/tcp               # Capture handwritten signatures
    ```

    打印文件内容的总行数。

    ```bash
    Shell > cat /etc/services | awk 'END{print NR}'
    11473
    ```

    打印第 200 行的内容。

    ```bash
    Shell > cat /etc/services | awk 'NR==200'
    microsoft-ds    445/tcp
    ```

    打印第 200 行的第二个字段。

    ```bash
    Shell > cat /etc/services | awk 'BEGIN{RS="\n";ORS="\n"} NR==200 {print $2}'
    445/tcp
    ```

    打印特定范围内的内容。

    ```bash
    Shell > cat /etc/services | awk 'BEGIN{RS="\n";ORS="\n"} NR<=10 {print NR,$0}'
    1 # /etc/services:
    2 # $Id: services,v 1.49 2017/08/18 12:43:23 ovasik Exp $
    3 #
    4 # Network services, Internet style
    5 # IANA services version: last updated 2016-07-08
    6 #
    7 # Note that it is presently the policy of IANA to assign a single well-known
    8 # port number for both TCP and UDP; hence, most entries here have two entries
    9 # even if the protocol doesn't support UDP operations.
    10 # Updated from RFC 1700, ``Assigned Numbers'' (October 1994).  Not all ports
    ```

    NR 与 FNR 的对比。

    ```bash
    Shell > head -n 3 /etc/services > /tmp/a.txt

    Shell > cat /tmp/a.txt
    # /etc/services:
    # $Id: services,v 1.49 2017/08/18 12:43:23 ovasik Exp $
    #

    Shell > cat /etc/resolv.conf
    # Generated by NetworkManager
    nameserver 8.8.8.8
    nameserver 114.114.114.114

    Shell > awk '{print NR,$0}' /tmp/a.txt /etc/resolv.conf
    1 # /etc/services:
    2 # $Id: services,v 1.49 2017/08/18 12:43:23 ovasik Exp $
    3 #
    4 # Generated by NetworkManager
    5 nameserver 8.8.8.8
    6 nameserver 114.114.114.114

    Shell > awk '{print FNR,$0}' /tmp/a.txt /etc/resolv.conf
    1 # /etc/services:
    2 # $Id: services,v 1.49 2017/08/18 12:43:23 ovasik Exp $
    3 #
    1 # Generated by NetworkManager
    2 nameserver 8.8.8.8
    3 nameserver 114.114.114.114
    ```

5. ARGC 和 ARGV

    ```bash
    Shell > awk 'BEGIN{print ARGC}' log dump long
    4
    Shell > awk 'BEGIN{print ARGV[0]}' log dump long
    awk
    Shell > awk 'BEGIN{print ARGV[1]}' log dump long
    log
    Shell > awk 'BEGIN{print ARGV[2]}' log dump long
    dump
    ```

6. ARGIND

    此变量主要用于判断 `awk` 程序正在处理的文件。

    ```bash
    Shell > awk '{print ARGIND,$0}' /etc/hostname /etc/resolv.conf
    1 Master
    2 # Generated by NetworkManager
    2 nameserver 8.8.8.8
    2 nameserver 114.114.114.114
    ```

7. ENVIRON

    你可以在 `awk` 程序中引用操作系统或用户定义的变量。

    ```bash
    Shell > echo ${SSH_CLIENT}
    192.168.100.2 6969 22

    Shell > awk 'BEGIN{print ENVIRON["SSH_CLIENT"]}'
    192.168.100.2 6969 22

    Shell > export a=123
    Shell > env | grep -w a
    a=123
    Shell > awk 'BEGIN{print ENVIRON["a"]}'
    123
    Shell > unset a
    ```

8. FILENAME

    ```bash
    Shell > awk 'BEGIN{RS="\n";ORS="\n"} NR=FNR {print ARGIND,FILENAME"---"$0}' /etc/hostname /etc/resolv.conf /etc/rocky-release
    1 /etc/hostname---Master
    2 /etc/resolv.conf---# Generated by NetworkManager
    2 /etc/resolv.conf---nameserver 8.8.8.8
    2 /etc/resolv.conf---nameserver 114.114.114.114
    3 /etc/rocky-release---Rocky Linux release 8.9 (Green Obsidian)
    ```

9. IGNORECASE

    如果你想在 `awk` 中使用正则表达式并忽略大小写，此变量非常有用。

    ```bash
    Shell > awk 'BEGIN{IGNORECASE=1;RS="\n";ORS="\n"} /^(SSH)|^(ftp)/ {print $0}' /etc/services
    ftp-data        20/tcp
    ftp-data        20/udp
    ftp             21/tcp
    ftp             21/udp          fsp fspd
    ssh             22/tcp                          # The Secure Shell (SSH) Protocol
    ssh             22/udp                          # The Secure Shell (SSH) Protocol
    ftp-data        20/sctp                 # FTP
    ftp             21/sctp                 # FTP
    ssh             22/sctp                 # SSH
    ftp-agent       574/tcp                 # FTP Software Agent System
    ftp-agent       574/udp                 # FTP Software Agent System
    sshell          614/tcp                 # SSLshell
    sshell          614/udp                 #       SSLshell
    ftps-data       989/tcp                 # ftp protocol, data, over TLS/SSL
    ftps-data       989/udp                 # ftp protocol, data, over TLS/SSL
    ftps            990/tcp                 # ftp protocol, control, over TLS/SSL
    ftps            990/udp                 # ftp protocol, control, over TLS/SSL
    ssh-mgmt        17235/tcp               # SSH Tectia Manager
    ssh-mgmt        17235/udp               # SSH Tectia Manager
    ```

    ```bash
    Shell > awk 'BEGIN{IGNORECASE=1;RS="\n";ORS="\n"} /^(SMTP)\s/,/^(TFTP)\s/ {print $0}' /etc/services
    smtp            25/tcp          mail
    smtp            25/udp          mail
    time            37/tcp          timserver
    time            37/udp          timserver
    rlp             39/tcp          resource        # resource location
    rlp             39/udp          resource        # resource location
    nameserver      42/tcp          name            # IEN 116
    nameserver      42/udp          name            # IEN 116
    nicname         43/tcp          whois
    nicname         43/udp          whois
    tacacs          49/tcp                          # Login Host Protocol (TACACS)
    tacacs          49/udp                          # Login Host Protocol (TACACS)
    re-mail-ck      50/tcp                          # Remote Mail Checking Protocol
    re-mail-ck      50/udp                          # Remote Mail Checking Protocol
    domain          53/tcp                          # name-domain server
    domain          53/udp
    whois++         63/tcp          whoispp
    whois++         63/udp          whoispp
    bootps          67/tcp                          # BOOTP server
    bootps          67/udp
    bootpc          68/tcp          dhcpc           # BOOTP client
    bootpc          68/udp          dhcpc
    tftp            69/tcp
    ```

## 运算符

| 运算符  | 说明                       |
|---------|----------------------------|
| (...)    | 分组                       |
| $n       | 字段引用                   |
| ++       | 自增                       |
| --       | 自减                       |
| +        | 数学加号                   |
| -        | 数学减号                   |
| !        | 取反                       |
| *        | 数学乘号                   |
| /        | 数学除号                   |
| %        | 取模运算                   |
| in       | 数组中的元素                 |
| &&       | 逻辑与运算                   |
| \|\|     | 逻辑或运算                   |
| ?:       | 条件表达式的缩写             |
| ~        | 正则表达式的另一种表示形式     |
| !~       | 反向正则表达式               |

!!! note

    在 `awk` 程序中，以下表达式将被判定为 **false**：

    * 数字为 0；
    * 空字符串；
    * 未定义的值。

    ```bash
    Shell > awk 'BEGIN{n=0;if(n) print "Ture";else print "False"}'
    False
    Shell > awk 'BEGIN{s="";if(s) print "True";else print "False"}'
    False
    Shell > awk 'BEGIN{if(t) print "True";else print "Flase"}'
    False
    ```

1. 感叹号

    打印奇数行：

    ```bash
    Shell > seq 1 10 | awk 'i=!i {print $0}'
    1
    3
    5
    7
    9
    ```

    !!! question

        **为什么？**
        **读取第一行**：因为 "i" 没有被赋值，所以 "i=!i" 表示 TRUE。
        **读取第二行**：此时，"i=!i" 表示 FALSE。
        依此类推，最终打印的行是奇数行。

    打印偶数行：

    ```bash
    Shell > seq 1 10 | awk '!(i=!i)'
    # 或
    Shell > seq 1 10 | awk '!(i=!i) {print $0}'
    2
    4
    6
    8
    10
    ```

    !!! note

        如你所见，有时可以忽略 "action" 部分的语法，默认情况下等价于 "{print $0}"。

2. 取反

    ```bash
    Shell > cat /etc/services | awk '!/(tcp)|(udp)|(^#)|(^$)/ {print $0}'
    http            80/sctp                         # HyperText Transfer Protocol
    bgp             179/sctp
    https           443/sctp                        # http protocol over TLS/SSL
    h323hostcall    1720/sctp                       # H.323 Call Control
    nfs             2049/sctp       nfsd shilp      # Network File System
    rtmp            1/ddp                           # Routing Table Maintenance Protocol
    nbp             2/ddp                           # Name Binding Protocol
    echo            4/ddp                           # AppleTalk Echo Protocol
    zip             6/ddp                           # Zone Information Protocol
    discard         9/sctp                  # Discard
    discard         9/dccp                  # Discard SC:DISC
    ...
    ```

3. 基本数学运算

    ```bash
    Shell > echo -e "36\n40\n50" | awk '{print $0+1}'
    37
    41

    Shell > echo -e "30\t5\t8\n11\t20\t34"
    30      5       8
    11      20      34
    Shell > echo -e "30\t5\t8\n11\t20\t34" | awk '{print $2*2+1}'
    11
    41
    ```

    也可以在 "pattern" 中使用：

    ```bash
    Shell > cat -n /etc/services | awk  '/^[1-9]*/ && $1%2==0 {print $0}'
    ...
    24  tcpmux          1/udp                           # TCP port service multiplexer
    26  rje             5/udp                           # Remote Job Entry
    28  echo            7/udp
    30  discard         9/udp           sink null
    32  systat          11/udp          users
    34  daytime         13/udp
    36  qotd            17/udp          quote
    ...

    Shell > cat -n /etc/services | awk  '/^[1-9]*/ && $1%2!=0 {print $0}'
    ...
    23  tcpmux          1/tcp                           # TCP port service multiplexer
    25  rje             5/tcp                           # Remote Job Entry
    27  echo            7/tcp
    29  discard         9/tcp           sink null
    31  systat          11/tcp          users
    ...
    ```

4. 管道符号

    你可以在 awk 程序中使用 bash 命令，例如：

    ```bash
    Shell > echo -e "6\n3\n9\n8" | awk '{print $0 | "sort"}'
    3
    6
    8
    9
    ```

    !!! info

        请注意！必须使用双引号包含命令。

5. 正则表达式

    [此处](#RE)介绍了正则表达式的基本示例。你可以对行记录使用正则表达式。

    ```bash
    Shell > cat /etc/services | awk '/[^0-9a-zA-Z]1[1-9]{2}\/tcp/ {print $0}'

    # 等价于：

    Shell > cat /etc/services | awk '$0~/[^0-9a-zA-Z]1[1-9]{2}\/tcp/ {print $0}'
    ```

    如果文件有大量文本，也可以对字段使用正则表达式，这有助于提高处理效率。用法示例如下：

    ```bash
    Shell > cat /etc/services | awk '$0~/^(ssh)/ && $2~/tcp/ {print $0}'
    ssh             22/tcp                          # The Secure Shell (SSH) Protocol
    sshell          614/tcp                 # SSLshell
    ssh-mgmt        17235/tcp               # SSH Tectia Manager

    Shell > cat /etc/services | grep -v -E "(^#)|(^$)" | awk '$2!~/(tcp)|(udp)/ {print $0}'
    http            80/sctp                         # HyperText Transfer Protocol
    bgp             179/sctp
    https           443/sctp                        # http protocol over TLS/SSL
    h323hostcall    1720/sctp                       # H.323 Call Control
    nfs             2049/sctp       nfsd shilp      # Network File System
    rtmp            1/ddp                           # Routing Table Maintenance Protocol
    nbp             2/ddp                           # Name Binding Protocol
    ...
    ```

## 流程控制

1. **if** 语句

    基本语法格式为 - `if (condition) statement [ else statement ]`

    if 语句单分支使用示例：

    ```bash
    Shell > cat /etc/services | awk '{if(NR==110) print $0}'
    pop3            110/udp         pop-3
    ```

    条件以正则表达式判定：

    ```bash
    Shell > cat /etc/services | awk '{if(/^(ftp)\s|^(ssh)\s/) print $0}'
    ftp             21/tcp
    ftp             21/udp          fsp fspd
    ssh             22/tcp                          # The Secure Shell (SSH) Protocol
    ssh             22/udp                          # The Secure Shell (SSH) Protocol
    ftp             21/sctp                 # FTP
    ssh             22/sctp                 # SSH
    ```

    双分支：

    ```bash
    Shell > seq 1 10 | awk '{if($0==10) print $0 ; else print "False"}'
    False
    False
    False
    False
    False
    False
    False
    False
    False
    10
    ```

    多分支：

    ```bash
    Shell > cat /etc/services | awk '{ \
    if($1~/netbios/)
        {print $0}
    else if($2~/175/)
        {print "175"}
    else if($2~/137/)
        {print "137"}
    else {print "no"}
    }'
    ```

2. **while** 语句

    基本语法格式为 - `while (condition) statement`

    遍历并打印所有行记录的字段。

    ```bash
    Shell > tail -n 2 /etc/services
    cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    spremotetablet  46998/tcp               # Capture handwritten signatures

    Shell > tail -n 2 /etc/services | awk '{ \
    i=1;
    while(i<=NF){print $i;i++}
    }'

    cloudcheck
    45514/tcp
    #
    ASSIA
    CloudCheck
    WiFi
    Management
    System
    spremotetablet
    46998/tcp
    #
    Capture
    handwritten
    signatures
    ```

3. **for** 语句

    基本语法格式为 - `for (expr1; expr2; expr3) statement`

    遍历并打印所有行记录的字段。

    ```bash
    Shell > tail -n 2 /etc/services | awk '{ \
    for(i=1;i<=NF;i++) print $i
    }'
    ```

    按逆序打印每行记录的字段。

    ```bash
    Shell > tail -n 2 /etc/services | awk '{ \
    for(i=NF;i>=1;i--) print $i
    }'

    System
    Management
    WiFi
    CloudCheck
    ASSIA
    #
    45514/tcp
    cloudcheck
    signatures
    handwritten
    Capture
    #
    46998/tcp
    spremotetablet
    ```

    按相反方向打印每行记录。

    ```bash
    Shell > tail -n 2 /etc/services | awk  '{ \
    for(i=NF;i>=1;i--) {printf $i" "};
    print ""
    }'

    System Management WiFi CloudCheck ASSIA # 45514/tcp cloudcheck
    signatures handwritten Capture # 46998/tcp spremotetablet
    ```

4. **break** 语句和 **continue** 语句<a id="bc"></a>

    两者的对比如下：

    ```bash
    Shell > awk 'BEGIN{  \
    for(i=1;i<=10;i++)
      {
        if(i==3) {break};
        print i
      }
    }'

    1
    2
    ```

    ```bash
    Shell > awk 'BEGIN{  \
    for(i=1;i<=10;i++)
      {
        if(i==3) {continue};
        print i
      }
    }'

    1
    2
    4
    5
    6
    7
    8
    9
    10
    ```

5. **exit** 语句

    可以指定一个 [0,255] 范围内的返回值。

    基本语法格式为 - `exit [expression]`

    ```bash
    Shell > seq 1 10 | awk '{
      if($0~/5/) exit "135"
    }'

    Shell > echo $?
    135
    ```

## 数组

**数组**：按一定顺序排列的具有相同数据类型的集合。数组中的每个数据称为一个元素。

像大多数编程语言一样，`awk` 也支持数组，分为 **索引数组（以数字为下标）** 和 **关联数组（以字符串为下标）**。

`awk` 有很多函数，与数组相关的函数有：

* **length(Array_Name)** - 获取数组的长度。

1. 自定义数组

    格式 - `Array_Name[Index]=Value`

    ```bash
    Shell > awk 'BEGIN{a1[0]="test0" ; a1[1]="s1"; print a1[0]}'
    test0
    ```

    获取数组的长度：

    ```bash
    Shell > awk 'BEGIN{name[-1]="jimcat8" ; name[3]="jack" ; print length(name)}'
    2
    ```

    将所有 GNU/Linux 用户存储在一个数组中：

    ```bash
    Shell > cat /etc/passwd | awk -F ":" '{username[NR]=$1}END{print username[2]}'
    bin
    Shell > cat /etc/passwd | awk -F ":" '{username[NR]=$1}END{print username[1]}'
    root
    ```

    !!! info

        `awk` 数组的数字下标可以是正整数、负整数、字符串或 0，因此 `awk` 数组的数字下标没有初始值的概念。这与 `bash` 中的数组不同。

        ```bash
        Shell > arr1=(2 10 30 string1)
        Shell > echo "${arr1[0]}"
        2
        Shell > unset arr1
        ```

2. 删除数组

    格式 - `delete Array_Name`

3. 删除数组中的某个元素

    格式 - `delete Array_Name[Index]`

4. 遍历数组

    你可以使用 **for** 语句，适用于数组下标未知的情况：

    ```bash
    Shell > head -n 5 /etc/passwd | awk -F ":" ' \
    {
      username[NR]=$1
    }
    END {
      for(i in username)
      print username[i],i
    }
    '

    root 1
    bin 2
    daemon 3
    adm 4
    lp 5
    ```

    如果数组的下标是有规律的，可以使用这种形式的 **for** 语句：

    ```bash
    Shell > cat /etc/passwd | awk -F ":" ' \
    {
      username[NR]=$1
    }
    END{
      for(i=1;i<=NR;i++)
      print username[i],i
    }
    '

    root 1
    bin 2
    daemon 3
    adm 4
    lp 5
    sync 6
    shutdown 7
    halt 8
    ...
    ```

5. 使用 "++" 作为数组的下标

    ```bash
    Shell > tail -n 5 /etc/group | awk -F ":" '\
    {
      a[x++]=$1
    }
    END{
      for(i in a)
      print a[i],i
    }
    '

    slocate 0
    unbound 1
    docker 2
    cgred 3
    redis 4
    ```

6. 使用字段作为数组的下标

    ```bash
    Shell > tail -n 5 /etc/group | awk -F ":" '\
    {
      a[$1]=$3
    }
    END{
      for(i in a)
      print a[i],i
    }
    '

    991 docker
    21 slocate
    989 redis
    992 unbound
    990 cgred
    ```

7. 统计相同字段出现的次数

    统计相同 IPv4 地址出现的次数。基本思路：

    * 首先使用 `grep` 命令过滤出所有 IPv4 地址
    * 然后交给 `awk` 程序处理

    ```bash
    Shell > cat /var/log/secure | egrep -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | awk ' \
    {
      a[$1]++
    }
    END{
      for(v in a) print a[v],v
    }
    '

    4 0.0.0.0
    4 192.168.100.2
    ```

    !!! info

        `a[$1]++` 等价于 `a[$1]+=1`

    统计不区分大小写的单词出现次数。基本思路：

    * 将所有字段分割成多行记录
    * 然后交给 `awk` 程序处理

    ```bash
    Shell > cat /etc/services | awk -F " " '{for(i=1;i<=NF;i++) print $i}'

    Shell > cat /etc/services | awk -F " " '{for(i=1;i<=NF;i++) print $i}' | awk '\
    BEGIN{IGNORECASE=1;OFS="\t"} /^netbios$/  ||  /^ftp$/  {a[$1]++}  END{for(v in a) print a[v],v}
    '

    3       NETBIOS
    18      FTP
    7       ftp

    Shell > cat /etc/services | awk -F " " '{ for(i=1;i<=NF;i++) print $i }' | awk '\
    BEGIN{IGNORECASE=1;OFS="\t"}  /^netbios$/  ||  /^ftp$/   {a[$1]++}  END{for(v in a)  \
    if(a[v]>=5) print a[v],v}
    '

    18      FTP
    7       ftp
    ```

    你可以先过滤特定的行记录，然后再进行统计，例如：

    ```bash
    Shell > ss -tulnp | awk -F " "  '/tcp/ {a[$2]++} END{for(i in a) print a[i],i}'
    2 LISTEN
    ```

8. 根据特定字段的出现次数打印行

    ```bash
    Shell > tail /etc/services
    aigairserver    21221/tcp               # Services for Air Server
    ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
    ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
    edi_service     34567/udp               # dhanalakshmi.org EDI Service
    axio-disc       35100/tcp               # Axiomatic discovery protocol
    axio-disc       35100/udp               # Axiomatic discovery protocol
    pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    spremotetablet  46998/tcp               # Capture handwritten signatures

    Shell > tail /etc/services | awk 'a[$1]++ {print $0}'
    axio-disc       35100/udp               # Axiomatic discovery protocol
    ```

    反向：

    ```bash
    Shell > tail /etc/services | awk '!a[$1]++ {print $0}'
    aigairserver    21221/tcp               # Services for Air Server
    ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
    ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
    edi_service     34567/udp               # dhanalakshmi.org EDI Service
    axio-disc       35100/tcp               # Axiomatic discovery protocol
    pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    spremotetablet  46998/tcp               # Capture handwritten signatures
    ```

9. 多维数组

    `awk` 程序不支持多维数组，但可以通过模拟来实现。默认情况下，"\034" 是多维数组下标的隔符。

    使用多维数组时请注意以下区别：

    ```bash
    Shell > awk 'BEGIN{ a["1,0"]=100 ; a[2,0]=200 ; a["3","0"]=300 ; for(i in a) print a[i],i }'
    200 20
    300 30
    100 1,0
    ```

    重新定义分隔符：

    ```bash
    Shell > awk 'BEGIN{ SUBSEP="----" ; a["1,0"]=100 ; a[2,0]=200 ; a["3","0"]=300 ; for(i in a) print a[i],i }'
    300 3----0
    200 2----0
    100 1,0
    ```

    重新排序：

    ```bash
    Shell > awk 'BEGIN{ SUBSEP="----" ; a["1,0"]=100 ; a[2,0]=200 ; a["3","0"]=300 ; for(i in a) print a[i],i | "sort" }'
    100 1,0
    200 2----0
    300 3----0
    ```

    统计字段出现次数：

    ```bash
    Shell > cat c.txt
    A 192.168.1.1 HTTP
    B 192.168.1.2 HTTP
    B 192.168.1.2 MYSQL
    C 192.168.1.1 MYSQL
    C 192.168.1.1 MQ
    D 192.168.1.4 NGINX

    Shell > cat c.txt | awk 'BEGIN{SUBSEP="----"} {a[$1,$2]++} END{for(i in a) print a[i],i}'
    1 A----192.168.1.1
    2 B----192.168.1.2
    2 C----192.168.1.1
    1 D----192.168.1.4
    ```

## 内置函数

| 函数名        | 说明                                                                                                                      |
| :---          | :---                                                                                                                     |
| int(expr)     | 截断为整数                                                                                                                 |
| sqrt(expr)    | 平方根                                                                                                                    |
| rand()        | 返回一个范围在 (0,1) 的随机数 N。结果不是每次运行都是随机数，而是保持不变。                                                       |
| srand([expr]) | 使用 "expr" 生成随机数。如果未指定 "expr"，默认使用当前时间作为种子，如果有种子，则使用生成的随机数。                                   |
| asort(a,b)    | 将数组 "a" 的元素重新排序（按字典序）并存储到新数组 "b" 中，数组 "b" 的下标从 1 开始。该函数返回数组中元素的数量。                       |
| asorti(a,b)   | 重新排序数组 "a" 的下标，并将排序后的下标作为元素存储到新数组 "b" 中，数组 "b" 的下标从 1 开始。                                       |
| sub(r,s[,t])  | 使用 "r" 正则表达式匹配输入记录，并用 "s" 替换匹配结果。"t" 是可选的，表示替换某个字段。该函数返回替换次数 - 0 或 1。类似于 `sed s//`          |
| gsub(r,s[,t]) | 全局替换。"t" 是可选的，表示替换某个字段。如果忽略 "t"，表示全局替换。类似于 `sed s///g`                                             |
| gensub(r,s,h[,t])  | "r" 正则表达式匹配输入记录，并用 "s" 替换匹配结果。"t" 是可选的，表示替换某个字段。"h" 表示替换指定索引位置                          |
| index(s,t)    | 返回字符串 "t" 在字符串 "s" 中的索引位置（字符串索引从 1 开始）。如果函数返回 0，表示不存在                                               |
| length([s])   | 返回 "s" 的长度                                                                                                           |
| match(s,r[,a])| 测试字符串 "s" 是否包含字符串 "r"。如果包含，返回 "r" 在其中的索引位置（字符串索引从 1 开始）。如果不包含，返回 0                                |
| split(s,a[,r[,seps]])| 基于分隔符 "seps" 将字符串 "s" 分割到数组 "a" 中。数组的下标从 1 开始。                                                            |
| substr(s,i[,n])  | 截取字符串。"s" 表示要处理的字符串；"i" 表示字符串的索引位置；"n" 是长度。如果不指定 "n"，表示截取所有剩余部分                            |
| tolower(str)  | 将所有字符串转换为小写                                                                                                     |
| toupper(str)  | 将所有字符串转换为大写                                                                                                     |
| systime()     | 当前时间戳                                                                                                                |
| strftime([format[,timestamp[,utc-flag]]]) | 格式化输出时间。将时间戳转换为字符串                                                 |

1. **int** 函数

    ```bash
    Shell > echo -e "qwer123\n123\nabc\n123abc123\n100.55\n-155.27"
    qwer123
    123
    abc
    123abc123
    100.55
    -155.27

    Shell > echo -e "qwer123\n123\nabc\n123abc123\n100.55\n-155.27" | awk '{print int($1)}'
    0
    123
    0
    123
    100
    -155
    ```

    如你所见，int 函数只对数字有效，遇到字符串时将其转换为 0。遇到以数字开头的字符串时，进行截断。

2. **sqrt** 函数

    ```bash
    Shell > awk 'BEGIN{print sqrt(9)}'
    3
    ```

3. **rand** 函数和 **srand** 函数

    rand 函数的使用示例如下：

    ```bash
    Shell > awk 'BEGIN{print rand()}'
    0.924046
    Shell > awk 'BEGIN{print rand()}'
    0.924046
    Shell > awk 'BEGIN{print rand()}'
    0.924046
    ```

    srand 函数的使用示例如下：

    ```bash
    Shell > awk 'BEGIN{srand() ; print rand()}'
    0.975495
    Shell > awk 'BEGIN{srand() ; print rand()}'
    0.99187
    Shell > awk 'BEGIN{srand() ; print rand()}'
    0.069002
    ```

    生成 (0,100) 范围内的整数：

    ```bash
    Shell > awk 'BEGIN{srand() ; print int(rand()*100)}'
    56
    Shell > awk 'BEGIN{srand() ; print int(rand()*100)}'
    33
    Shell > awk 'BEGIN{srand() ; print int(rand()*100)}'
    42
    ```

4. **asort** 函数和 **asorti** 函数

    ```bash
    Shell > cat /etc/passwd | awk -F ":" '{a[NR]=$1} END{anu=asort(a,b) ; for(i=1;i<=anu;i++) print i,b[i]}'
    1 adm
    2 bin
    3 chrony
    4 daemon
    5 dbus
    6 ftp
    7 games
    8 halt
    9 lp
    10 mail
    11 nobody
    12 operator
    13 polkitd
    14 redis
    15 root
    16 shutdown
    17 sshd
    18 sssd
    19 sync
    20 systemd-coredump
    21 systemd-resolve
    22 tss
    23 unbound

    Shell > awk 'BEGIN{a[1]=1000 ; a[2]=200 ; a[3]=30 ; a[4]="admin" ; a[5]="Admin" ; \
    a[6]="12string" ; a[7]=-1 ; a[8]=-10 ; a[9]=-20 ; a[10]=-21 ;nu=asort(a,b) ; for(i=1;i<=nu;i++) print i,b[i]}'
    1 -21
    2 -20
    3 -10
    4 -1
    5 30
    6 200
    7 1000
    8 12string
    9 Admin
    10 admin
    ```

    !!! info

        排序规则：

        * 数字优先级高于字符串，按升序排列。
        * 字符串按字典升序排列

    如果使用 **asorti** 函数，示例如下：

    ```bash
    Shell > awk 'BEGIN{ a[-11]=1000 ; a[-2]=200 ; a[-10]=30 ; a[-21]="admin" ; a[41]="Admin" ; \
    a[30]="12string" ; a["root"]="rootstr" ; a["Root"]="r1" ; nu=asorti(a,b) ; for(i in b) print i,b[i] }'
    1 -10
    2 -11
    3 -2
    4 -21
    5 30
    6 41
    7 Root
    8 root
    ```

    !!! info

        排序规则：

        * 数字优先级高于字符串
        * 如果遇到负数，从左边的第一位开始比较。如果相同，则比较第二位，依此类推
        * 如果遇到正数，按升序排列
        * 字符串按字典升序排列

5. **sub** 函数和 **gsub** 函数

    ```bash
    Shell > cat /etc/services | awk '/netbios/ {sub(/tcp/,"test") ; print $0 }'
    netbios-ns      137/test                         # NETBIOS Name Service
    netbios-ns      137/udp
    netbios-dgm     138/test                         # NETBIOS Datagram Service
    netbios-dgm     138/udp
    netbios-ssn     139/test                         # NETBIOS session service
    netbios-ssn     139/udp

    Shell > cat /etc/services |  awk '/^ftp/ && /21\/tcp/  {print $0}'
    ftp             21/tcp
      ↑                  ↑
    Shell > cat /etc/services |  awk 'BEGIN{OFS="\t"}  /^ftp/ && /21\/tcp/   {gsub(/p/,"P",$2) ; print $0}'
    ftp     21/tcP
                 ↑
    Shell > cat /etc/services |  awk 'BEGIN{OFS="\t"}  /^ftp/ && /21\/tcp/   {gsub(/p/,"P") ; print $0}'
    ftP             21/tcP
      ↑                  ↑
    ```

    与 `sed` 命令一样，你也可以使用 "&" 符号来引用已匹配的字符串。

    ```bash
    Shell > vim /tmp/tmp-file1.txt
    A 192.168.1.1 HTTP
    B 192.168.1.2 HTTP
    B 192.168.1.2 MYSQL
    C 192.168.1.1 MYSQL
    C 192.168.1.1 MQ
    D 192.168.1.4 NGINX

    # 在第二行之前添加一行文本
    Shell > cat /tmp/tmp-file1.txt | awk 'NR==2 {gsub(/.*/,"add a line\n&")} {print $0}'
    A 192.168.1.1 HTTP
    add a line
    B 192.168.1.2 HTTP
    B 192.168.1.2 MYSQL
    C 192.168.1.1 MYSQL
    C 192.168.1.1 MQ
    D 192.168.1.4 NGINX

    # 在第二行 IP 地址后添加字符串
    Shell > cat /tmp/tmp-file1.txt | awk 'NR==2 {gsub(/[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}/,"&\tSTRING")} {print $0}'
    A 192.168.1.1 HTTP
    B 192.168.1.2   STRING HTTP
    B 192.168.1.2 MYSQL
    C 192.168.1.1 MYSQL
    C 192.168.1.1 MQ
    D 192.168.1.4 NGINX
    ```

6. **index** 函数

    ```bash
    Shell > tail -n 5 /etc/services
    axio-disc       35100/udp               # Axiomatic discovery protocol
    pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    spremotetablet  46998/tcp               # Capture handwritten signatures

    Shell > tail -n 5 /etc/services | awk '{print index($2,"tcp")}'
    0
    7
    0
    7
    7
    ```

7. **length** 函数

    ```bash
    # 输出字段的长度
    Shell > tail -n 5 /etc/services | awk '{print length($1)}'
    9
    8
    15
    10
    14

    # 输出数组的长度
    Shell > cat /etc/passwd | awk -F ":" 'a[NR]=$1 END{print length(a)}'
    22
    ```

8. **match** 函数

    ```bash
    Shell > echo -e "1592abc144qszd\n144bc\nbn"
    1592abc144qszd
    144bc
    bn

    Shell > echo -e "1592abc144qszd\n144bc\nbn" | awk '{print match($1,144)}'
    8
    1
    0
    ```

9. **split** 函数

    ```bash
    Shell > echo "365%tmp%dir%number" | awk '{split($1,a1,"%") ; for(i in a1) print i,a1[i]}'
    1 365
    2 tmp
    3 dir
    4 number
    ```

10. **substr** 函数

    ```bash
    Shell > head -n 5 /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

    # 我需要这部分内容 - "emon:/sbin:/sbin/nologin"
    Shell > head -n 5 /etc/passwd | awk '/daemon/ {print substr($0,16)}'
    emon:/sbin:/sbin/nologin

    Shell > tail -n 5 /etc/services
    axio-disc       35100/udp               # Axiomatic discovery protocol
    pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    spremotetablet  46998/tcp               # Capture handwritten signatures

    # 我需要这部分内容 - "tablet"
    Shell > tail  -n 5 /etc/services | awk '/^sp/ {print substr($1,9)}'
    tablet
    ```

11. **tolower** 函数和 **toupper** 函数

    ```bash
    Shell > echo -e "AbcD123\nqweR" | awk '{print tolower($0)}'
    abcd123
    qwer

    Shell > tail -n 5 /etc/services | awk '{print toupper($0)}'
    AXIO-DISC       35100/UDP               # AXIOMATIC DISCOVERY PROTOCOL
    PMWEBAPI        44323/TCP               # PERFORMANCE CO-PILOT CLIENT HTTP API
    CLOUDCHECK-PING 45514/UDP               # ASSIA CLOUDCHECK WIFI MANAGEMENT KEEPALIVE
    CLOUDCHECK      45514/TCP               # ASSIA CLOUDCHECK WIFI MANAGEMENT SYSTEM
    SPREMOTETABLET  46998/TCP               # CAPTURE HANDWRITTEN SIGNATURES
    ```

12. 处理时间和日期的函数

    **什么是 UNIX 时间戳？**
    根据 GNU/Linux 的发展历史，UNIX V1 诞生于 1971 年，同年 11 月 3 日出版了《UNIX 程序员手册》，该手册将 1970-01-01 定义为 UNIX 开始的参考日期。

    时间戳与自然日期时间之间以天为单位的转换：

    ```bash
    Shell > echo "$(( $(date --date="2024/01/06" +%s)/86400 + 1 ))"
    19728

    Shell > date -d "1970-01-01 19728days"
    Sat Jan  6 00:00:00 CST 2024
    ```

    时间戳与自然日期时间之间以秒为单位的转换：

    ```bash
    Shell > echo "$(date --date="2024/01/06 17:12:00" +%s)"
    1704532320

    Shell > echo "$(date --date='@1704532320')"
    Sat Jan  6 17:12:00 CST 2024
    ```

    `awk` 程序中自然日期时间与 UNIX 时间戳之间的转换：

    ```bash
    Shell > awk 'BEGIN{print systime()}'
    1704532597

    Shell > echo "1704532597" | awk '{print strftime("%Y-%m-%d %H:%M:%S",$0)}'
    2024-01-06 17:16:37
    ```

## I/O 语句

| 语句                      | 说明                                                                                                            |
| :---                      | :---                                                                                                           |
| getline                   | 读取下一条匹配的行记录并将其赋值给 "$0"。返回值为 1：表示已读取到相关行记录。返回值为 0：表示已读取到最后一行。返回值为负数：表示遇到错误。            |
| getline var               | 读取下一条匹配的行记录并将其赋值给变量 "var"                                                                                 |
| command \| getline [var]  | 将结果赋值给 "$0" 或变量 "var"                                                                                  |
| next                      | 停止当前输入记录并执行后续动作                                                                                       |
| print                     | 打印结果                                                                                                        |
| printf                    | 参见本文档中该命令的章节                                                                                             |
| system(cmd-line)          | 执行命令并返回状态码。0 表示命令执行成功；非 0 表示执行失败                                                               |
| print ... >> file         | 输出重定向                                                                                                      |
| print ... \| command      | 打印输出并将其用作命令的输入                                                                                        |

1. getline

    ```bash
    Shell > seq 1 10 | awk '/3/ || /6/ {getline ; print $0}'
    4
    7

    Shell > seq 1 10 | awk '/3/ || /6/ {print $0 ; getline ; print $0}'
    3
    4
    6
    7
    ```

    使用我们之前学过的函数和 "&" 符号，我们可以：

    ```bash
    Shell > tail -n 5 /etc/services | awk '/45514\/tcp/ {getline ; gsub(/.*/ , "&\tSTRING1") ; print $0}'
    spremotetablet  46998/tcp               # Capture handwritten signatures        STRING1

    Shell > tail -n 5 /etc/services | awk '/45514\/tcp/ {print $0 ; getline; gsub(/.*/,"&\tSTRING2") } {print $0}'
    axio-disc       35100/udp               # Axiomatic discovery protocol
    pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    spremotetablet  46998/tcp               # Capture handwritten signatures        STRING2
    ```

    打印偶数和奇数行：

    ```bash
    Shell > tail -n 10 /etc/services | cat -n | awk '{ if( (getline) <= 1) print $0}'
    2  ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
    4  edi_service     34567/udp               # dhanalakshmi.org EDI Service
    6  axio-disc       35100/udp               # Axiomatic discovery protocol
    8  cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    10  spremotetablet  46998/tcp               # Capture handwritten signatures

    Shell > tail -n 10 /etc/services | cat -n | awk '{if(NR==1) print $0} { if(NR%2==0) {if(getline > 0) print $0} }'
    1  aigairserver    21221/tcp               # Services for Air Server
    3  ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
    5  axio-disc       35100/tcp               # Axiomatic discovery protocol
    7  pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    9  cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    ```

2. getline var

    将 b 文件的每一行添加到 C 文件每一行的末尾：

    ```bash
    Shell > cat /tmp/b.txt
    b1
    b2
    b3
    b4
    b5
    b6

    Shell > cat /tmp/c.txt
    A 192.168.1.1 HTTP
    B 192.168.1.2 HTTP
    B 192.168.1.2 MYSQL
    C 192.168.1.1 MYSQL
    C 192.168.1.1 MQ
    D 192.168.1.4 NGINX

    Shell > awk '{getline var1 <"/tmp/b.txt" ; print $0 , var1}' /tmp/c.txt
    A 192.168.1.1 HTTP b1
    B 192.168.1.2 HTTP b2
    B 192.168.1.2 MYSQL b3
    C 192.168.1.1 MYSQL b4
    C 192.168.1.1 MQ b5
    D 192.168.1.4 NGINX b6
    ```

    用 b 文件的内容替换 c 文件的指定字段：

    ```bash
    Shell > awk '{ getline var2 < "/tmp/b.txt" ; gsub($2 , var2 , $2) ; print $0 }' /tmp/c.txt
    A b1 HTTP
    B b2 HTTP
    B b3 MYSQL
    C b4 MYSQL
    C b5 MQ
    D b6 NGINX
    ```

3. command | getline &#91;var&#93;

    ```bash
    Shell > awk 'BEGIN{ "date +%Y%m%d" | getline datenow ; print datenow}'
    20240107
    ```

    !!! tip

        使用双引号包含 Shell 命令。

4. next

    之前我们介绍了 **break** 语句和 **continue** 语句，前者用于终止循环，后者用于跳出当前循环。参见[此处](#bc)。对于 **next**，当条件满足时，它将停止满足条件的输入记录，并继续执行后续动作。

    ```bash
    Shell > seq 1 5 | awk '{if(NR==3) {next} print $0}'
    1
    2
    4
    5

    # 等价于
    Shell > seq 1 5 | awk '{if($1!=3) print $0}'
    ```

    跳过符合条件的行记录：

    ```bash
    Shell > cat /etc/passwd | awk -F ":" 'NR>5 {next} {print $0}'
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

    # 等价于
    Shell > cat /etc/passwd | awk -F ":" 'NR>=1 && NR<=5 {print $0}'
    ```

    !!! tip

        "**next**" 不能在 "BEGIN{}" 和 "END{}" 中使用。

5. **system** 函数

    你可以使用该函数调用 Shell 中的命令，例如：

    ```bash
    Shell > awk 'BEGIN{ system("echo nginx http") }'
    nginx http
    ```

    !!! tip

        请注意在使用 **system** 函数时添加双引号。如果不添加，`awk` 程序会将其视为 `awk` 程序的变量。

        ```bash
        Shell > awk 'BEGIN{ cmd1="date +%Y" ; system(cmd1)}'
        2024
        ```

    **如果 Shell 命令本身包含双引号怎么办？**
    使用转义字符 - "\\"，例如：

    ```bash
    Shell > egrep "^root|^nobody" /etc/passwd
    Shell > awk 'BEGIN{ system("egrep \"^root|^nobody\" /etc/passwd") }'
    root:x:0:0:root:/root:/bin/bash
    nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
    ```

    另一个例子：

    ```bash
    Shell > awk 'BEGIN{ if ( system("xmind &> /dev/null") == 0 ) print "True"; else print "False" }'
    False
    ```

6. 将 `awk` 程序的输出写入文件

    ```bash
    Shell > head -n 5 /etc/passwd | awk -F ":" 'BEGIN{OFS="\t"} {print $1,$2 > "/tmp/user.txt"}'
    Shell > cat /tmp/user.txt
    root    x
    bin     x
    daemon  x
    adm     x
    lp      x
    ```

    !!! tip

        "**>**" 表示以覆盖方式写入文件。如果要以追加方式写入文件，请使用 "**>>**"。再次提醒，应使用双引号包含文件路径。

7. 管道符号

8. 自定义函数

    语法 - `function NAME(parameter list) { function body }`。例如：

    ```bash
    Shell > awk 'function mysum(a,b) {return a+b} BEGIN{print mysum(1,6)}'
    7
    ```

## 总结性说明

如果你具有专业的编程语言技能，`awk` 相对容易学习。然而，对于大多数编程语言技能较弱的系统管理员（包括作者本人），`awk` 的学习可能会非常复杂。更多信息请参考[此处](https://www.gnu.org/software/gawk/manual/ "gawk manual")。

再次感谢你的阅读。
