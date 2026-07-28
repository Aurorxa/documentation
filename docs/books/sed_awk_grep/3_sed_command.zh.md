---
title: Sed 命令
author: tianci li
contributors: 
tags:
  - sed
---

# `sed` 命令

`sed`：流编辑器（Stream Editor）

**工作原理**：`sed` 命令会读取当前处理的行并将其放入"pattern space"中进行处理。处理完成后，将结果输出并清空"pattern space"。接着读取下一行并放入"pattern space"中处理，依此类推，直到最后一行。有些文档还提到一个称为"hold space"的术语，它可以临时存储一些已处理的数据，并通过"pattern space"输出。

**"pattern space"和"hold space"**：内存中用于处理和存储数据的区域。

更多信息请参考 [`sed` 手册页](https://www.gnu.org/software/sed/manual/)。

命令用法为：

```bash
sed [OPTION]... {script-only-if-no-other-script} [input-file]...
```

| 选项 | 说明 |
| :---: | :--- |
| -n    | 只将经过 `sed` 命令处理过的文本行输出到屏幕 |
| -e    | 对输入的文本行数据应用多个 `sed` 操作命令 |
| -f    | 调用并执行 `sed` 脚本命令文件 |
| -i    | 修改原始文件 |
| -r    | 正则表达式 |

| 操作命令 | 说明 |
| :---:                  | :--- |
| s/regexp/replacement/  | 替换字符串 |
|  p                     | 打印当前"pattern space"。常与 -n 选项配合使用，例如：`cat -n /etc/services \| sed -n '3,5p'` |
|  d                     | 删除"pattern space"。开始下一个循环    |
|  D                     | 删除"pattern space"的第一行，开始下一个循环 |
|  =                     | 打印行号 |
| a \text                | 在匹配行后添加一行或多行内容。添加多行时，除最后一行外的所有行都需要使用"\"表示内容尚未结束 |
| i \text                | 在匹配行前添加一行或多行内容。添加多行时，除最后一行外的所有行都需要使用"\"表示内容尚未结束 |
| c \text                | 用新文本替换匹配行 |
| q                      | 立即退出 `sed` 脚本    |
| r                      | 追加从文件读取的文本 |
| : label                | b 和 t 命令的标签 |
| b label                | 跳转到标签处；如果省略标签，则跳转到脚本末尾 |
| t label                | 如果"s///"替换成功，则跳转到标签 |
| h H                    | 复制/追加"pattern space"到"hold space"  |
| g G                    | 复制/追加"hold space"到"pattern space"  |
| x                      | 交换 hold space 和 pattern space 的内容 |
| l                      | 以"视觉明确"的形式列出当前行 |
| n N                    | 读取/追加下一行输入到"pattern space" |
| w FILENAME             | 将当前"pattern space"写入 FILENAME       |
| !                      | 取反 |
| &                      | 引用已匹配的字符串 |

| 地址表示法    | 说明 |
| :---:        |  :---       |
| first~step   | 用"first"指定第一行，用"step"指定步长。例如，使用 `sed -n "1~2p" /etc/services` 输出奇数行文本 |
| $            | 匹配最后一行文本 |
| /regexp/     | 使用正则表达式匹配文本行 |
| number       | 指定行号  |
| addr1,addr2  | 使用行号定位匹配从"addr1"到"addr2"的所有行 |
| addr1,+N     | 使用行号定位匹配"addr1"及其后的 N 行 |

## 使用示例

1. 匹配并打印（`p`）

    * 打印以 netbios 字符串开头的行

        ```bash
        Shell > cat /etc/services | sed -n '/^netbios/p'
        netbios-ns      137/tcp                         # NETBIOS Name Service
        netbios-ns      137/udp
        netbios-dgm     138/tcp                         # NETBIOS Datagram Service
        netbios-dgm     138/udp
        netbios-ssn     139/tcp                         # NETBIOS session service
        netbios-ssn     139/udp
        ```

        !!! tip

            众所周知，shell 中的双引号和单引号起着不同的作用。双引号中的 **$**、**\`** 和 **\\** 具有特殊含义。建议在使用 `sed` 命令时多使用单引号。

    * 打印第 23 到 26 行的文本

        ```bash
        Shell > cat -n /etc/services | sed -n '23,26p'
        23  tcpmux          1/tcp                           # TCP port service multiplexer
        24  tcpmux          1/udp                           # TCP port service multiplexer
        25  rje             5/tcp                           # Remote Job Entry
        26  rje             5/udp                           # Remote Job Entry
        ```

    * 打印奇数行

        ```bash
        Shell > cat -n /etc/services | sed -n '1~2p'
        1  # /etc/services:
        3  #
        5  # IANA services version: last updated 2016-07-08
        7  # Note that it is presently the policy of IANA to assign a single well-known
        9  # even if the protocol doesn't support UDP operations.
        11  # are included, only the more common ones.
        13  # The latest IANA port assignments can be gotten from
        15  # The Well Known Ports are those from 0 through 1023.
        17  # The Dynamic and/or Private Ports are those from 49152 through 65535
        19  # Each line describes one service, and is of the form:
        ...
        ```

    * 打印第 10 行到最后一行

        ```bash
        Shell > cat -n /etc/services | sed -n '10,$p'
        10  # Updated from RFC 1700, ``Assigned Numbers'' (October 1994).  Not all ports
        11  # are included, only the more common ones.
        12  #
        13  # The latest IANA port assignments can be gotten from
        14  #       http://www.iana.org/assignments/port-numbers
        15  # The Well Known Ports are those from 0 through 1023.
        16  # The Registered Ports are those from 1024 through 49151
        17  # The Dynamic and/or Private Ports are those from 49152 through 65535
        ...
        ```

    * 第 10 行到最后一行不打印

        ```bash
        Shell > cat -n /etc/services | sed -n '10,$!p'
        1  # /etc/services:
        2  # $Id: services,v 1.49 2017/08/18 12:43:23 ovasik Exp $
        3  #
        4  # Network services, Internet style
        5  # IANA services version: last updated 2016-07-08
        6  #
        7  # Note that it is presently the policy of IANA to assign a single well-known
        8  # port number for both TCP and UDP; hence, most entries here have two entries
        9  # even if the protocol doesn't support UDP operations.
        ```

    * 打印匹配字符串所在的行号及内容

        ```bash
        Shell > sed -n -e '/netbios/=' -e '/netbios/p' /etc/services
        123
        netbios-ns      137/tcp                         # NETBIOS Name Service
        124
        netbios-ns      137/udp
        125
        netbios-dgm     138/tcp                         # NETBIOS Datagram Service
        126
        netbios-dgm     138/udp
        127
        netbios-ssn     139/tcp                         # NETBIOS session service
        128
        netbios-ssn     139/udp
        ```

    * 匹配字符串范围并打印

        使用逗号分隔字符串范围

        ```bash
        Shell > cat  /etc/services | sed -n '/^netbios/,/^imap/p'
        netbios-ns      137/tcp                         # NETBIOS Name Service
        netbios-ns      137/udp
        netbios-dgm     138/tcp                         # NETBIOS Datagram Service
        netbios-dgm     138/udp
        netbios-ssn     139/tcp                         # NETBIOS session service
        netbios-ssn     139/udp
        imap            143/tcp         imap2           # Interim Mail Access Proto v2
        ```

        !!! info

            **范围起始**：匹配字符串所在的行，只匹配第一个出现的字符串。
            **范围结束**：匹配字符串所在的行，只匹配第一个出现的字符串。

        ```bash
        Shell > grep -n ^netbios /etc/services
        123:netbios-ns      137/tcp                         # NETBIOS Name Service
        124:netbios-ns      137/udp
        125:netbios-dgm     138/tcp                         # NETBIOS Datagram Service
        126:netbios-dgm     138/udp
        127:netbios-ssn     139/tcp                         # NETBIOS session service
        128:netbios-ssn     139/udp

        Shell > grep -n ^imap /etc/services
        129:imap            143/tcp         imap2           # Interim Mail Access Proto v2
        130:imap            143/udp         imap2
        168:imap3           220/tcp                         # Interactive Mail Access
        169:imap3           220/udp                         # Protocol v3
        260:imaps           993/tcp                         # IMAP over SSL
        261:imaps           993/udp                         # IMAP over SSL
        ```

        也就是说，上面打印的内容是第 123 行到第 129 行

    * 打印字符串所在的行直到最后一行

        ```bash
        Shell > cat /etc/services | sed -n '/^netbios/,$p'
        ```

    * 在 bash 脚本中使用变量

        ```bash
        Shell > vim test1.sh
        #!/bin/bash
        a=10

        sed -n ''${a}',$!p' /etc/services
        # 或
        sed -n "${a},\$!p" /etc/services
        ```

    * 正则表达式

        只匹配"三位数字" + "/udp"。

        ```bash
        Shell > cat /etc/services  | sed -r -n '/\s([1-9][0-9]{2}\/udp)/p' | sort -k 2
        hostname        101/udp         hostnames       # usually from sri-nic
        iso-tsap        102/udp                 # ISO-TSAP Class 0
        gppitnp         103/udp                 # Genesis Point-to-Point Trans Net
        acr-nema        104/udp                 # ACR-NEMA Digital Imag. & Comm. 300
        csnet-ns        105/udp         cso
        poppassd        106/udp                         # Eudora
        #3com-tsmux     106/udp         poppassd
        rtelnet         107/udp
        snagas          108/udp                 # SNA Gateway Access Server
        ...
        ```

2. 匹配并删除（`d`）

    与打印类似，只是操作命令替换为 `d`，且不需要 -n 选项。

    * 删除所有匹配 udp 字符串的行、删除所有注释行和删除所有空行

        ```bash
        Shell > sed -e '/udp/d' -e '/^#/d' -e '/^$/d' /etc/services
        tcpmux          1/tcp                           # TCP port service multiplexer
        rje             5/tcp                           # Remote Job Entry
        echo            7/tcp
        discard         9/tcp           sink null
        systat          11/tcp          users
        daytime         13/tcp
        qotd            17/tcp          quote
        chargen         19/tcp          ttytst source
        ftp-data        20/tcp
        ftp             21/tcp
        ssh             22/tcp                          # The Secure Shell (SSH) Protocol
        telnet          23/tcp
        ...
        ```

    * 删除连续行文本

        ```bash
        Shell > cat -n /etc/services | sed '10,$d'
        1  # /etc/services:
        2  # $Id: services,v 1.49 2017/08/18 12:43:23 ovasik Exp $
        3  #
        4  # Network services, Internet style
        5  # IANA services version: last updated 2016-07-08
        6  #
        7  # Note that it is presently the policy of IANA to assign a single well-known
        8  # port number for both TCP and UDP; hence, most entries here have two entries
        9  # even if the protocol doesn't support UDP operations.
        ```

    * 正则表达式

        ```bash
        Shell > cat  /etc/services | sed -r '/(tcp)|(udp)|(^#)|(^$)/d'
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

3. 替换字符串（`s///g`）

    | 语法                                 | 语法说明                                                                                                                                                                                                                |
    | :---                                 | :---                                                                                                                                                                                                                  |
    | `sed 's/string/replace/g' FILENAME`  | **s**：代表文件内容的所有行。也可以指定行范围，例如：`sed '20,200s/netbios/TMP/g' /etc/services`。**g**（Global）：如果没有 g，表示当一行中出现多个匹配字符串时，只替换第一个匹配的字符串。**/**：分隔符风格。也可以指定其他风格，例如：`sed '20,200s?netbios?TMP?g' /etc/services` |

    你也可以在 bash 脚本中使用替换功能。

    例如：

    ```bash
    Shell > vim /root/sedReplace.sh
    #!/bin/bash
    a="SELINUX=enforcing"
    b="SELINUX=disabled"

    sed -i 's/'${a}'/'${b}'/g' /etc/selinux/config
    # 或
    sed -i "s/${a}/${b}/g" /etc/selinux/config
    ```

    * 替换并打印

        ```bash
        Shell > sed -n '44,45s/ssh/SSH/gp' /etc/services
        SSH             22/tcp
        SSH             22/udp
        ```

    * 使用 "&" 符号引用字符串

        ```bash
        Shell > sed -n '44,45s/ssh/&-SSH/gp' /etc/services
        ssh-SSH             22/tcp
        ssh-SSH             22/udp
        ```

    * 使用字符串定位一行或多行，并替换行范围内指定的字符串

        ```bash
        Shell > grep ssh /etc/services -n
        44:ssh             22/tcp                          # The Secure Shell (SSH) Protocol
        45:ssh             22/udp                          # The Secure Shell (SSH) Protocol
        551:x11-ssh-offset  6010/tcp                        # SSH X11 forwarding offset
        593:ssh             22/sctp                 # SSH
        1351:sshell          614/tcp                 # SSLshell
        1352:sshell          614/udp                 #       SSLshell
        1607:netconf-ssh     830/tcp                 # NETCONF over SSH
        1608:netconf-ssh     830/udp                 # NETCONF over SSH
        7178:sdo-ssh         3897/tcp                # Simple Distributed Objects over SSH
        7179:sdo-ssh         3897/udp                # Simple Distributed Objects over SSH
        7791:netconf-ch-ssh  4334/tcp                # NETCONF Call Home (SSH)
        8473:snmpssh         5161/tcp                # SNMP over SSH Transport Model
        8474:snmpssh-trap    5162/tcp                # SNMP Notification over SSH Transport Model
        9126:tl1-ssh         6252/tcp                # TL1 over SSH
        9127:tl1-ssh         6252/udp                # TL1 over SSH
        10796:ssh-mgmt        17235/tcp               # SSH Tectia Manager
        10797:ssh-mgmt        17235/udp               # SSH Tectia Manager

        Shell > sed '/ssh/s/tcp/TCP/gp' -n  /etc/services
        ssh             22/TCP                          # The Secure Shell (SSH) Protocol
        x11-ssh-offset  6010/TCP                        # SSH X11 forwarding offset
        sshell          614/TCP                 # SSLshell
        netconf-ssh     830/TCP                 # NETCONF over SSH
        sdo-ssh         3897/TCP                # Simple Distributed Objects over SSH
        netconf-ch-ssh  4334/TCP                # NETCONF Call Home (SSH)
        snmpssh         5161/TCP                # SNMP over SSH Transport Model
        snmpssh-trap    5162/TCP                # SNMP Notification over SSH Transport Model
        tl1-ssh         6252/TCP                # TL1 over SSH
        ssh-mgmt        17235/TCP               # SSH Tectia Manager
        ```

    * 连续行的字符串替换

        ```bash
        Shell > sed '10,30s/tcp/TCP/g' /etc/services
        ```

    * 多次匹配和替换

        ```bash
        Shell > cat /etc/services | sed 's/netbios/test1/g ; s/^#//d ; s/dhcp/&t2/g'
        ```

    * 正则表达式的分组替换

        在正则表达式中，"()" 是一个分组。\1 表示引用第 1 组，\2 表示引用第 2 组，依此类推。

        ```bash
        Shell > cat /etc/services
        ...
        axio-disc       35100/tcp               # Axiomatic discovery protocol
        axio-disc       35100/udp               # Axiomatic discovery protocol
        pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
        cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
        cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        spremotetablet  46998/tcp               # Capture handwritten signatures

        Shell > cat /etc/services | sed -r 's/([0-9]*\/tcp)/\1\tCONTENT1/g ; s/([0-9]*\/udp)/\1\tADD2/g'
        ...
        axio-disc       35100/tcp       CONTENT1               # Axiomatic discovery protocol
        axio-disc       35100/udp       ADD2               # Axiomatic discovery protocol
        pmwebapi        44323/tcp       CONTENT1               # Performance Co-Pilot client HTTP API
        cloudcheck-ping 45514/udp       ADD2               # ASSIA CloudCheck WiFi Management keepalive
        cloudcheck      45514/tcp       CONTENT1               # ASSIA CloudCheck WiFi Management System
        spremotetablet  46998/tcp       CONTENT1               # Capture handwritten signatures
        ```

        **\t**：即制表符

    * 将所有注释行替换为空行

        ```bash
        Shell > cat /etc/services | sed -r 's/(^#.*)//g'
        ...
        chargen         19/udp          ttytst source
        ftp-data        20/tcp
        ftp-data        20/udp

        ftp             21/tcp
        ftp             21/udp          fsp fspd
        ssh             22/tcp                          # The Secure Shell (SSH) Protocol
        ssh             22/udp                          # The Secure Shell (SSH) Protocol
        ...
        ```

    * 将单词中的一个小写字母替换为大写字母

        ```bash
        Shell > echo -e "hello,world\nPOSIX" | sed -r 's/(.*)w/\1W/g'
        hello,World
        POSIX
        ```

    * 字符串位置交换

         ```bash
         Shell > cat /etc/services
         ...
         cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
         cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
         spremotetablet  46998/tcp               # Capture handwritten signatures
         ```

         我们可以将此文件分为五个部分：

         ```txt
         cloudcheck-ping    45514       /     udp        # ASSIA CloudCheck WiFi Management keepalive
          ↓                   ↓         ↓      ↓               ↓
         (.*)           (\<[0-9]+\>)   \/   (tcp|udp)         (.*)
          ↓                   ↓                ↓               ↓ 
          \1                 \2               \3              \4
         ```

        ```bash
        Shell > cat /etc/services | sed -r 's/(.*)(\<[0-9]+\>)\/(tcp|udp)(.*)/\1\3\/\2\4/g'
        ...
        edi_service     udp/34567               # dhanalakshmi.org EDI Service
        axio-disc       tcp/35100               # Axiomatic discovery protocol
        axio-disc       udp/35100               # Axiomatic discovery protocol
        pmwebapi        tcp/44323               # Performance Co-Pilot client HTTP API
        cloudcheck-ping udp/45514               # ASSIA CloudCheck WiFi Management keepalive
        cloudcheck      tcp/45514               # ASSIA CloudCheck WiFi Management System
        spremotetablet  tcp/46998               # Capture handwritten signatures
        ```

    * 删除所有空白字符

        ```bash
        Shell > echo -e "abcd\t1 2 3 4\tWorld"
        abcd    1 2 3 4 World
        Shell > echo -e "abcd\t1 2 3 4\tWorld" | sed -r 's/(\s)*//g'
        abcd1234World
        ```

4. 使用 -e 选项执行多次操作

    示例如下：

    ```bash
    Shell > tail -n 10 /etc/services
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

    Shell > tail -n 10 /etc/services | sed  -e '1,3d' -e '/cloud/s/ping/PING/g'
    # 或
    Shell > tail -n 10 /etc/services | sed  '1,3d ; /cloud/s/ping/PING/g'
    edi_service     34567/udp               # dhanalakshmi.org EDI Service
    axio-disc       35100/tcp               # Axiomatic discovery protocol
    axio-disc       35100/udp               # Axiomatic discovery protocol
    pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    cloudcheck-PING 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    spremotetablet  46998/tcp               # Capture handwritten signatures
    ```

5. 在特定行上方或下方添加内容（`i` 和 `a`）

    * 在指定行号上方添加两行内容

        ```bash
        Shell > tail -n 10 /etc/services > /root/test.txt
        Shell > cat /root/test.txt
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

        Shell > cat /root/test.txt | sed '3i 123\
        abc'
        aigairserver    21221/tcp               # Services for Air Server
        ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
        123
        abc
        ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
        edi_service     34567/udp               # dhanalakshmi.org EDI Service
        axio-disc       35100/tcp               # Axiomatic discovery protocol
        axio-disc       35100/udp               # Axiomatic discovery protocol
        pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
        cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
        cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        spremotetablet  46998/tcp               # Capture handwritten signatures
        ```

    * 在指定行号下方添加三行内容

        ```bash
        Shell > cat /root/test.txt | sed '5a 123\
        comment yes\
        tcp or udp'
        aigairserver    21221/tcp               # Services for Air Server
        ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
        ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
        edi_service     34567/udp               # dhanalakshmi.org EDI Service
        axio-disc       35100/tcp               # Axiomatic discovery protocol
        123
        comment yes
        tcp or udp
        axio-disc       35100/udp               # Axiomatic discovery protocol
        pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
        cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
        cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        spremotetablet  46998/tcp               # Capture handwritten signatures
        ```

    * 根据字符串匹配特定行，并在其上方添加 2 行内容

        ```bash
        Shell > cat /root/test.txt | sed '/tcp/iTCP\
        UDP'
        TCP
        UDP
        aigairserver    21221/tcp               # Services for Air Server
        ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
        TCP
        UDP
        ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
        edi_service     34567/udp               # dhanalakshmi.org EDI Service
        TCP
        UDP
        axio-disc       35100/tcp               # Axiomatic discovery protocol
        axio-disc       35100/udp               # Axiomatic discovery protocol
        TCP
        UDP
        pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
        cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
        TCP
        UDP
        cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        TCP
        UDP
        spremotetablet  46998/tcp               # Capture handwritten signatures
        ```

6. 替换行（`c`）

    * 根据字符串定位一行或多行，并替换这些文本行

        ```bash
        Shell > cat /root/test.txt | sed '/ser/c\TMP1 \
        TMP2'
        TMP1
        TMP2
        ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
        ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
        TMP1
        TMP2
        axio-disc       35100/tcp               # Axiomatic discovery protocol
        axio-disc       35100/udp               # Axiomatic discovery protocol
        pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
        cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
        cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        spremotetablet  46998/tcp               # Capture handwritten signatures
        ```

    * 单行替换

        ```bash
        Shell > cat /root/test.txt | sed '7c REPLACE'
        aigairserver    21221/tcp               # Services for Air Server
        ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
        ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
        edi_service     34567/udp               # dhanalakshmi.org EDI Service
        axio-disc       35100/tcp               # Axiomatic discovery protocol
        axio-disc       35100/udp               # Axiomatic discovery protocol
        REPLACE
        cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
        cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        spremotetablet  46998/tcp               # Capture handwritten signatures
        ```

    * 替换连续行文本

        ```bash
        Shell > cat /root/test.txt | sed '2,$c REPLACE1 \
        replace2'
        aigairserver    21221/tcp               # Services for Air Server
        REPLACE1
        replace2
        ```

    * 替换偶数行

        ```bash
        Shell > cat /root/test.txt | sed '2~2c replace'
        aigairserver    21221/tcp               # Services for Air Server
        replace
        ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
        replace
        axio-disc       35100/tcp               # Axiomatic discovery protocol
        replace
        pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
        replace
        cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        replace
        ```

7. 读取文件内容并将其追加到匹配行下方（`r`）

    ```bash
    Shell > cat /root/app.txt
    append1
    POSIX
    UNIX

    Shell > cat /root/test.txt | sed '/ping/r /root/app.txt'
    aigairserver    21221/tcp               # Services for Air Server
    ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
    ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
    edi_service     34567/udp               # dhanalakshmi.org EDI Service
    axio-disc       35100/tcp               # Axiomatic discovery protocol
    axio-disc       35100/udp               # Axiomatic discovery protocol
    pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
    cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
    append1
    POSIX
    UNIX
    cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
    spremotetablet  46998/tcp               # Capture handwritten signatures
    ```

8. 将匹配行写入其他文件（`w`）

    ```bash
    Shell > cat /root/test.txt | sed '/axio/w /root/storage.txt'

    Shell > cat /root/storage.txt
    axio-disc       35100/tcp               # Axiomatic discovery protocol
    axio-disc       35100/udp               # Axiomatic discovery protocol
    ```

9. 将下一行输入读取/追加到"pattern space"中（`n` 和 `N`）

    * 打印匹配行的下一行

        ```bash
        Shell > cat /root/test.txt
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

        Shell > cat /root/test.txt | sed '/ping/{n;p}' -n
        cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        ```

        !!! tip

            多个 `sed` 操作命令可能会相互影响，你可以使用 "**{ }**" 来减少这种影响。

    * 打印偶数文本行

        首先读取第一行。由于使用了 `n` 命令，将打印第二行，依此类推。

        ```bash
        Shell > cat -n /root/test.txt | sed -n '{n;p}'
        # 或
        Shell > cat -n /root/test.txt | sed -n '2~2p'
        2  ka-kdp          31016/udp               # Kollective Agent Kollective Delivery
        4  edi_service     34567/udp               # dhanalakshmi.org EDI Service
        6  axio-disc       35100/udp               # Axiomatic discovery protocol
        8  cloudcheck-ping 45514/udp               # ASSIA CloudCheck WiFi Management keepalive
        10  spremotetablet  46998/tcp               # Capture handwritten signatures
        ```

    * 打印奇数文本行

        ```bash
        Shell > cat -n /root/test.txt | sed -n '{p;n}'
        # 或
        Shell > cat -n /root/test.txt | sed -n '1~2p'
        # 或
        Shell > cat -n /root/test.txt | sed 'n;d'
        1  aigairserver    21221/tcp               # Services for Air Server
        3  ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
        5  axio-disc       35100/tcp               # Axiomatic discovery protocol
        7  pmwebapi        44323/tcp               # Performance Co-Pilot client HTTP API
        9  cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        ```

    * 打印 3n 行

        ```bash
        Shell > cat -n /root/test.txt | sed -n '{n;n;p}'
        # 或
        Shell > cat -n /root/test.txt | sed -n '3~3p'
        3  ka-sddp         31016/tcp               # Kollective Agent Secure Distributed Delivery
        6  axio-disc       35100/udp               # Axiomatic discovery protocol
        9  cloudcheck      45514/tcp               # ASSIA CloudCheck WiFi Management System
        ```

    * `N`

        读取第一行，遇到 `N` 命令后追加一行。在此示例中，"pattern space" 是 "1\n2"。最后执行 `q` 命令退出。

        ```bash
        Shell > seq 1 10 | sed 'N;q'
        1
        2
        ```

        因为第 9 行之后没有额外的行，所以输出如下：

        ```bash
        Shell > seq 1 9 | sed -n 'N;p'
        1
        2
        3
        4
        5
        6
        7
        8
        ```

        当读取最后一行时，`N` 命令不会被执行，输出如下：

        ```bash
        Shell > seq 1 9 | sed -n '$!N;p'
        1
        2
        3
        4
        5
        6
        7
        8
        9
        ```

        将两行合并为一行。将"pattern space"的"\n"替换为空白字符。

        ```bash
        Shell > seq 1 6 | sed 'N;{s/\n//g}'
        12
        34
        56
        ```

10. 忽略大小写（`I`）

    `man 1 sed` 中似乎没有关于忽略大小写的信息。

    ```bash
    Shell > echo -e "abc\nAbc" | sed -n 's/a/X/Igp'
    Xbc
    XBC
    ```

    ```bash
    Shell > cat /etc/services | sed '/OEM/Ip' -n
    oem-agent       3872/tcp                # OEM Agent
    oem-agent       3872/udp                # OEM Agent
    oemcacao-jmxmp  11172/tcp               # OEM cacao JMX-remoting access point
    oemcacao-rmi    11174/tcp               # OEM cacao rmi registry access point
    oemcacao-websvc 11175/tcp               # OEM cacao web service access point
    ```

    ```bash
    Shell > cat /etc/services | sed -r '/(TCP)|(UDP)/Id'
    ```

    ```bash
    Shell > cat /etc/services | sed -r '/(TCP)|(UDP)/Ic TMP'
    ```

11. 获取文件的总行数

    ```bash
    Shell > cat /etc/services | sed -n '$='
    # 或
    Shell > cat /etc/services | wc -l

    11473
    ```
