---
title: dump 和 restore 命令
author: tianci li
contributors: Steven Spencer 
tested_with: 8.10
tags:
  - dump
  - restore
  - backup
---

## 概述

`dump` 检查文件系统中的文件，确定哪些需要备份，并将这些文件复制到指定的磁盘、磁带或其他存储介质。`restore` 命令执行 `dump` 的反向功能。

此工具适用于以下文件系统：

* ext2
* ext3
* ext4

!!! tip

    对于 xfs 文件系统，请使用 `xfsdump`。

[这里](https://dump.sourceforge.io/)是项目的主页。

在使用此工具之前，运行以下命令进行安装：

```bash
dnf -y install dump
```

安装后，有两个常用命令工具可用：

* `dump`
* `restore`

### `dump` 命令

此命令有两个主要用途：

* 执行备份（dump）操作 - `dump [option(s)] -f <File-Name> <File1> ...`
* 查看备份（dump）信息 - `dump [-W | -w]`

常用选项有：

* `-<level>` - 备份级别。使用时请将 "level" 替换为 0-9 之间的任意数字。数字 0 表示全量备份，其他数字表示增量备份。
* `-f <File-Name>` - 指定备份后的文件名和路径。
* `-u` - 备份成功后，将备份时间记录在 **/etc/dumpdates** 文件中。当备份对象是独立分区时可以使用 `-u` 选项。但是，当备份对象是非分区目录时不能使用 `-u` 选项。
* `-v` - 在备份过程中显示处理详情。
* `-W` - 查看 dump 信息的选项。
* `-z[LEVEL]` - 使用 zlib 库调整压缩级别，默认压缩级别为 2。例如，您可以将备份文件压缩为 `.gz` 格式。压缩级别的可调范围是 1-9。
* `-j[LEVEL]` - 使用 bzlib 库调整压缩级别，默认压缩级别为 2。例如，您可以将备份文件压缩为 `.bz2` 格式。压缩级别的可调范围是 1-9。

#### 使用 `dump` 的示例

1. 对根分区执行全量备份：

    ```bash
    dump -0u -j3 -f /tmp/root-20241208.bak.bz2 /
    DUMP: Date of this level 0 dump: Sun Dec  8 19:04:39 2024
    DUMP: Dumping /dev/nvme0n1p2 (/) to /tmp/root-20241208.bak.bz2
    DUMP: Label: none
    DUMP: Writing 10 Kilobyte records
    DUMP: Compressing output at transformation level 3 (bzlib)
    DUMP: mapping (Pass I) [regular files]
    DUMP: mapping (Pass II) [directories]
    DUMP: estimated 14693111 blocks.
    DUMP: Volume 1 started with block 1 at: Sun Dec  8 19:04:41 2024
    DUMP: dumping (Pass III) [directories]
    DUMP: dumping (Pass IV) [regular files]
    DUMP: 20.69% done at 10133 kB/s, finished in 0:19
    DUMP: 43.74% done at 10712 kB/s, finished in 0:12
    DUMP: 70.91% done at 11575 kB/s, finished in 0:06
    DUMP: 93.23% done at 11415 kB/s, finished in 0:01
    DUMP: Closing /tmp/root-20241208.bak.bz2
    DUMP: Volume 1 completed at: Sun Dec  8 19:26:08 2024
    DUMP: Volume 1 took 0:21:27
    DUMP: Volume 1 transfer rate: 5133 kB/s
    DUMP: Volume 1 14722930kB uncompressed, 6607183kB compressed, 2.229:1
    DUMP: 14722930 blocks (14377.86MB) on 1 volume(s)
    DUMP: finished in 1287 seconds, throughput 11439 kBytes/sec
    DUMP: Date of this level 0 dump: Sun Dec  8 19:04:39 2024
    DUMP: Date this dump completed:  Sun Dec  8 19:26:08 2024
    DUMP: Average transfer rate: 5133 kB/s
    DUMP: Wrote 14722930kB uncompressed, 6607183kB compressed, 2.229:1
    DUMP: DUMP IS DONE

    ls -lh /tmp/root-20241208.bak.bz2
    -rw-r--r-- 1 root root 6.4G Dec  8 19:26 /tmp/root-20241208.bak.bz2
    ```

2. dump 成功后，检查相关信息：

    ```bash
    cat /etc/dumpdates
    /dev/nvme0n1p2 0 Sun Dec  8 19:04:39 2024 +0800

    dump -W
    Last dump(s) done (Dump '>' file systems):
    /dev/nvme0n1p2        (     /) Last dump: Level 0, Date Sun Dec  8 19:04:39 2024
    ```

3. 在全量备份的基础上执行增量备份：

    ```bash
    echo "jack" >> /tmp/tmpfile.txt

    dump -1u -j4 -f /tmp/root-20241208-LV1.bak.bz2 /
    DUMP: Date of this level 1 dump: Sun Dec  8 19:38:51 2024
    DUMP: Date of last level 0 dump: Sun Dec  8 19:04:39 2024
    DUMP: Dumping /dev/nvme0n1p2 (/) to /tmp/root-20241208-LV1.bak.bz2
    DUMP: Label: none
    DUMP: Writing 10 Kilobyte records
    DUMP: Compressing output at transformation level 4 (bzlib)
    DUMP: mapping (Pass I) [regular files]
    DUMP: mapping (Pass II) [directories]
    DUMP: estimated 6620898 blocks.
    DUMP: Volume 1 started with block 1 at: Sun Dec  8 19:38:58 2024
    DUMP: dumping (Pass III) [directories]
    DUMP: dumping (Pass IV) [regular files]
    DUMP: 38.13% done at 8415 kB/s, finished in 0:08
    DUMP: 75.30% done at 8309 kB/s, finished in 0:03
    DUMP: Closing /tmp/root-20241208-LV1.bak.bz2
    DUMP: Volume 1 completed at: Sun Dec  8 19:52:03 2024
    DUMP: Volume 1 took 0:13:05
    DUMP: Volume 1 transfer rate: 8408 kB/s
    DUMP: Volume 1 6620910kB uncompressed, 6600592kB compressed, 1.004:1
    DUMP: 6620910 blocks (6465.73MB) on 1 volume(s)
    DUMP: finished in 785 seconds, throughput 8434 kBytes/sec
    DUMP: Date of this level 1 dump: Sun Dec  8 19:38:51 2024
    DUMP: Date this dump completed:  Sun Dec  8 19:52:03 2024
    DUMP: Average transfer rate: 8408 kB/s
    DUMP: Wrote 6620910kB uncompressed, 6600592kB compressed, 1.004:1
    DUMP: DUMP IS DONE

    cat /etc/dumpdates
    /dev/nvme0n1p2 0 Sun Dec  8 19:04:39 2024 +0800
    /dev/nvme0n1p2 1 Sun Dec  8 19:38:51 2024 +0800

    dump -W
    Last dump(s) done (Dump '>' file systems):
    /dev/nvme0n1p2        (     /) Last dump: Level 1, Date Sun Dec  8 19:38:51 2024
    ```

4. 对于非分区目录，只能使用全量备份（`-0`）选项，不能使用 `-u` 选项：

    ```bash
    dump -0uj -f /tmp/etc-full-20241208.bak.bz2 /etc/
    DUMP: You can't update the dumpdates file when dumping a subdirectory
    DUMP: The ENTIRE dump is aborted.

    dump -0j -f /tmp/etc-full-20241208.bak.bz2 /etc/
    DUMP: Date of this level 0 dump: Sun Dec  8 20:00:38 2024
    DUMP: Dumping /dev/nvme0n1p2 (/ (dir etc)) to /tmp/etc-full-20241208.bak.bz2
    DUMP: Label: none
    DUMP: Writing 10 Kilobyte records
    DUMP: Compressing output at transformation level 2 (bzlib)
    DUMP: mapping (Pass I) [regular files]
    DUMP: mapping (Pass II) [directories]
    DUMP: estimated 28204 blocks.
    DUMP: Volume 1 started with block 1 at: Sun Dec  8 20:00:38 2024
    DUMP: dumping (Pass III) [directories]
    DUMP: dumping (Pass IV) [regular files]
    DUMP: Closing /tmp/etc-full-20241208.bak.bz2
    DUMP: Volume 1 completed at: Sun Dec  8 20:00:40 2024
    DUMP: Volume 1 took 0:00:02
    DUMP: Volume 1 transfer rate: 3751 kB/s
    DUMP: Volume 1 29090kB uncompressed, 7503kB compressed, 3.878:1
    DUMP: 29090 blocks (28.41MB) on 1 volume(s)
    DUMP: finished in 2 seconds, throughput 14545 kBytes/sec
    DUMP: Date of this level 0 dump: Sun Dec  8 20:00:38 2024
    DUMP: Date this dump completed:  Sun Dec  8 20:00:40 2024
    DUMP: Average transfer rate: 3751 kB/s
    DUMP: Wrote 29090kB uncompressed, 7503kB compressed, 3.878:1
    DUMP: DUMP IS DONE
    ```

    对 /etc/ 目录执行增量备份将导致错误：

    ```bash
    dump -1j -f /tmp/etc-incr-20241208.bak.bz2 /etc/
    DUMP: Only level 0 dumps are allowed on a subdirectory
    DUMP: The ENTIRE dump is aborted.
    ```

### `restore` 命令

此命令的用法是 - `restore <mode(flag)> [option(s)] -f <Dump-File>`

模式（标志）可以是以下之一：

* `-C` - 比较模式。Restore 读取备份并将其内容与磁盘上的文件进行比较。它主要用于对分区执行备份后进行比较。在此模式下，`restore` 仅比较基于原始数据的更改。如果磁盘上有新数据，则无法比较或检测到。
* `-i` - 交互模式。此模式允许从 dump 中交互式地恢复文件。
* `-t` - 列表模式。列出备份文件中有哪些数据。
* `-r` - 恢复（重建）模式。如果是"全量备份 + 增量备份"方式，恢复数据将按时间顺序进行。
* `-x` - 提取模式。从备份文件中提取部分或全部文件。

#### 使用 `restore` 的示例

1. 从 /tmp/etc-full-20241208.bak.bz2 恢复数据：

    ```bash
    mkdir /tmp/data/

    restore -t -f /tmp/etc-full-20241208.bak.bz2

    cd /tmp/data/ ; restore -r -f /tmp/etc-full-20241208.bak.bz2

    ls -l /tmp/data/
    total 4992
    drwxr-xr-x. 90 root root    4096 Dec  8 17:13 etc
    -rw-------   1 root root 5107632 Dec  8 20:39 restoresymtable
    ```

    如您所见，成功恢复后会出现一个名为 `restoresymtable` 的文件。这个文件很重要。它用于增量备份系统恢复操作。

2. 在交互模式下处理备份文件：

    ```bash
    restore -i -f /tmp/etc-full-20241208.bak.bz2
    Dump tape is compressed.
    
    restore > ?
    ```

    您可以输入 ++question++ 来查看此模式下可用的交互式命令。
