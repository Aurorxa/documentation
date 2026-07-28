---
title: tar 命令
author: tianci li
contributors: Ganna Zhyrnova, Steven Spencer
tested_with: 8.10
tags:
  - tar
  - backup
  - archive
---

## 概述

`tar` 是 GNU/Linux 和其他 UNIX 操作系统上用于处理归档文件的工具。它代表 "tape archive"（磁带归档）。

将文件方便地存储在磁带上是最初使用 tar 归档的方式。"tar" 的名字由此而来。尽管该工具有其名称渊源，`tar` 可以将其输出定向到可用的设备、文件或其他程序（使用管道），并访问远程设备或文件（作为归档）。

现代 GNU/Linux 上当前使用的 `tar` 最初来自 [GNU 项目](https://www.gnu.org/)。您可以在 [GNU 的网站](https://ftp.gnu.org/gnu/tar/)浏览和下载所有版本的 `tar`。

!!! note

    不同发行版中的 `tar` 可能有不同的默认选项。请在使用时注意。

    ```bash
    # RockyLinux 8 和 Fedora 41
    tar --show-defaults
    --format=gnu -f- -b20 --quoting-style=escape --rmt-command=/etc/rmt --rsh-command=/usr/bin/ssh
    ```

## 使用 `tar`

使用 `tar` 时，请注意它有两种保存模式：

* **相对模式**（默认）：移除文件的前导字符 '/'。即使您以绝对路径添加文件，`tar` 在此模式下也会移除前导字符 "/"。
* **绝对模式**：保留前导字符 '/' 并将其包含在文件名中。您需要使用 `-P` 选项来启用此保存模式。在此模式下，您必须将所有文件表示为绝对路径。出于安全原因，除非有特殊场景需求，在大多数情况下不应使用此保存模式。

当您使用 `tar` 时，会遇到 `.tar.gz`、`.tar.xz` 和 `.tar.bz2` 等后缀，这表明您先创建了一个归档（将相关文件归类为单个文件），然后用相应的压缩类型或算法对该文件进行压缩。

压缩类型或算法可以是 gzip、bzip2、xz、zstd 或其他。

`tar` 允许从备份中提取单个文件或目录、查看其内容或验证其完整性。

创建归档并使用压缩的用法是：

* `tar [option] [PATH] [DIR1] ... [FILE1] ...`。例如，`tar -czvf /tmp/Fullbackup-20241201.tar.gz /etc/ /var/log/`

从归档中提取文件的用法是：

* `tar [option] [PATH] -C [dir]`。例如 `tar -xzvf /tmp/Fullbackup-20241201.tar.gz -C /tmp/D1`

!!! tip

     当从归档文件中提取文件时，`tar` 会根据手动添加的后缀自动选择压缩类型。例如，对于 `.tar.gz` 文件，可以直接使用 `tar -vxf` 而无需使用 `tar -zvxf`。对于创建归档压缩文件，**必须**选择压缩类型。

!!! Note

    在 GNU/Linux 中，除桌面环境（GUI）需要之外，大多数文件不需要扩展名。人为添加后缀是为了方便用户识别。例如，如果系统管理员看到一个 `.tar.gz` 或 `.tgz` 文件扩展名，他就知道如何处理该文件。

### 操作参数或类型

| 类型 | 描述 |
| :---: | :--- |
| `-A`  | 将一个归档中的所有文件追加到另一个归档的末尾。仅适用于 `.tar` 类型的非压缩归档文件 |
| `-c`  | 创建归档。非常常用 |
| `-d`  | 比较归档文件与对应未归档文件之间的差异 |
| `-r`  | 将文件或目录追加到归档末尾。仅适用于 `.tar` 类型的非压缩归档文件 |
| `-t`  | 列出归档内容 |
| `-u`  | 仅将较新的文件追加到归档中。仅适用于 `.tar` 类型的非压缩归档文件 |
| `-x`  | 从归档中提取。非常常用 |
| `--delete` | 从 ".tar" 归档中删除文件或目录。仅适用于 `.tar` 类型的非压缩归档文件 |

!!! Tip

    作者建议保留前缀 "-" 以保持用户对操作类型的使用习惯。这并非必需。此处的操作参数指示您使用 `tar` 的主要功能。换句话说，您需要从上述类型中选择一个。

### 常用辅助选项

| 选项 | 描述                                                                                             |
|--------|---------------------------------------------------------------------------------------------------------|
| `-z`   | 使用 `gzip` 作为其压缩类型。创建归档和从归档中提取均适用  |
| `-v`   | 显示详细处理信息                                                                    |
| `-f`   | 指定归档的文件名（包括文件后缀）                                            |
| `-j`   | 使用 `bzip2` 作为其压缩类型。创建归档和从归档中提取均适用 |
| `-J`   | 使用 `xz` 作为其压缩类型。创建归档和从归档中提取均适用    |
| `-C`   | 从归档中提取文件后的保存位置                                                                           |
| `-P`   | 使用绝对模式保存                                                                           |

其他未提及的辅助选项，请参见 `man 1 tar`

!!! warning "版本差异"

    在某些旧版本的 tar 中，选项被称为 "key(s)"，这意味着使用带有 "-" 前缀的选项可能导致 `tar` 无法按预期工作。此时，您需要移除 "-" 前缀以使其正常工作。

### 关于选项风格

`tar` 提供三种选项风格：

1. 传统风格：

    * `tar {A|c|d|r|t|u|x}[GnSkUWOmpsMBiajJzZhPlRvwo] [ARG...]`。

2. 短选项风格的用法：

    * `tar -A [OPTIONS] ARCHIVE ARCHIVE`
    * `tar -c [-f ARCHIVE] [OPTIONS] [FILE...]`
    * `tar -d [-f ARCHIVE] [OPTIONS] [FILE...]`
    * `tar -t [-f ARCHIVE] [OPTIONS] [MEMBER...]`
    * `tar -r [-f ARCHIVE] [OPTIONS] [FILE...]`
    * `tar -u [-f ARCHIVE] [OPTIONS] [FILE...]`
    * `tar -x [-f ARCHIVE] [OPTIONS] [MEMBER...]`

3. 长选项风格的用法：

    * `tar {--catenate|--concatenate} [OPTIONS] ARCHIVE ARCHIVE`
    * `tar --create [--file ARCHIVE] [OPTIONS] [FILE...]`
    * `tar {--diff|--compare} [--file ARCHIVE] [OPTIONS] [FILE...]`
    * `tar --delete [--file ARCHIVE] [OPTIONS] [MEMBER...]`
    * `tar --append [-f ARCHIVE] [OPTIONS] [FILE...]`
    * `tar --list [-f ARCHIVE] [OPTIONS] [MEMBER...]`
    * `tar --test-label [--file ARCHIVE] [OPTIONS] [LABEL...]`
    * `tar --update [--file ARCHIVE] [OPTIONS] [FILE...]`
    * `tar --update [-f ARCHIVE] [OPTIONS] [FILE...]`
    * `tar {--extract|--get} [-f ARCHIVE] [OPTIONS] [MEMBER...]`

第二种方法是大多数 GNU/Linux 用户更常用的。

### 压缩效率和使用频率

`tar` 本身没有压缩能力，因此必须与其他压缩工具配合使用。压缩和解压缩会影响资源消耗。

以下是对一组文本文件压缩效率从低到高的排名：

* compress (`.tar.Z`) - 较少使用
* gzip (`.tar.gz` 或 `.tgz`) - 常用
* bzip2 (`.tar.bz2` 或 `.tb2` 或 `.tbz`) - 常用
* lzip (`.tar.lz`) - 较少使用
* xz (`.tar.xz`) - 常用

### `tar` 的命名惯例

以下是 `tar` 归档文件命名惯例的示例：

| 主要功能和辅助选项 | 文件 | 后缀 | 功能 |
|--------  |---------|------------------|----------------------------------------------|
| `-cvf`   | `home`  | `home.tar`       | `/home` 以相对模式，无压缩形式 |
| `-cvfP`  | `/etc`  | `etc.A.tar`      | `/etc` 以绝对模式，无压缩 |
| `-cvfz`  | `usr`   | `usr.tar.gz`     | `/usr` 以相对模式，*gzip* 压缩 |
| `-cvfj`  | `usr`   | `usr.tar.bz2`    | `/usr` 以相对模式，*bzip2* 压缩 |
| `-cvfPz` | `/home` | `home.A.tar.gz`  | `/home` 以绝对模式，*gzip* 压缩 |
| `-cvfPj` | `/home` | `home.A.tar.bz2` | `/home` 以绝对模式，*bzip2* 压缩 |

您也可以将日期添加到文件名中。

### 使用示例

#### `-c` 类型

1. 以相对模式归档并压缩 **/etc/**，后缀为 `.tar.gz`：

    ```bash
    tar -czvf /tmp/etc-20241207.tar.gz /etc/
    ```

    由于 `tar` 默认以相对模式工作，命令输出的第一行将显示以下内容：

    ```bash
    tar: Removing leading '/' from member names
    ```

2. 归档 **/var/log/** 并选择 xz 类型进行压缩：

    ```bash
    tar -cJvf /tmp/log-20241207.tar.xz /var/log/

    du -sh /var/log/ ; ls -lh /tmp/log-20241207.tar.xz
    18M     /var/log/
    -rw-r--r-- 1 root root 744K Dec  7 14:40 /tmp/log-20241207.tar.xz
    ```

3. 估算文件大小而不生成归档：

    ```bash
    tar -cJf - /etc | wc -c
    tar: Removing leading `/' from member names
    3721884
    ```

    `wc -c` 命令的输出单位是字节。

4. 切割大型 `.tar.gz` 文件：

    ```bash
    cd /tmp/ ; tar -czf - /etc/  | split -d -b 2M - etc-backup20241207.tar.gz.

    ls -lh /tmp/
    -rw-r--r-- 1 root root 2.0M Dec  7 20:46 etc-backup20241207.tar.gz.00
    -rw-r--r-- 1 root root 2.0M Dec  7 20:46 etc-backup20241207.tar.gz.01
    -rw-r--r-- 1 root root 2.0M Dec  7 20:46 etc-backup20241207.tar.gz.02
    -rw-r--r-- 1 root root  70K Dec  7 20:46 etc-backup20241207.tar.gz.03
    ```

    第一个 "-" 代表 `tar` 的输入参数，第二个 "-" 告诉 `tar` 将输出重定向到 `stdout`。

    要提取这些切割的小文件，可以执行以下操作：

    ```bash
    cd /tmp/ ; cat etc-backup20241207.tar.gz.* >> /tmp/etc-backup-20241207.tar.gz

    cd /tmp/ ; tar -xvf etc-backup-20241207.tar.gz -C /tmp/dir1/
    ```

#### `-x` 类型

1. 下载 Redis 源代码并提取到 `/usr/local/src/` 目录：

    ```bash
    wget -c https://github.com/redis/redis/archive/refs/tags/7.4.1.tar.gz

    tar -xvf 7.4.1.tar.gz -C /usr/local/src/
    ```

2. 仅从归档压缩文件中提取一个文件：

    ```bash
    tar -xvf /tmp/etc-20241207.tar.gz etc/chrony.conf
    ```

#### `-A` 或 `-r` 类型

1. 将一个 `.tar` 文件追加到另一个 `.tar` 文件：

    ```bash
    tar -cvf /tmp/etc.tar /etc/

    tar -cvf /tmp/log.tar /var/log/

    tar -Avf /tmp/etc.tar /tmp/log.tar
    ```

    这意味着 "log.tar" 中的所有文件将追加到 "etc.tar" 的末尾。

2. 将文件或目录追加到 `.tar` 文件：

    ```bash
    tar -rvf /tmp/log.tar /etc/chrony.conf
    tar: Removing leading `/' from member names
    /etc/chrony.conf
    tar: Removing leading `/' from hard link targets
    
    tar -rvf /tmp/log.tar /tmp/dir1
    ```

!!! warning

    无论使用 `-A` 还是 `-r` 选项，都要考虑相关归档文件的保存模式。

!!! warning

    `-A` 和 `-r` 不适用于归档压缩文件。

#### `-t` 类型

1. 查看归档内容：

    ```bash
    tar -tvf /tmp/log.tar

    tar -tvf /tmp/etc-20241207.tar.gz | less
    ```

#### `-d` 类型

1. 比较文件差异：

    ```bash
    cd / ; tar -dvf /tmp/etc.tar etc/chrony.conf
    etc/chrony.conf

    cd / ; tar -dvf /tmp/etc-20241207.tar.gz etc/
    ```

    对于使用相对模式的存储方式，使用 `-d` 类型时，将文件路径切换到 '/'。

#### `-u` 类型

1. 如果同一文件有多个版本，可以使用 `-u` 类型：

    ```bash
    touch /tmp/tmpfile1

    tar -rvf /tmp/log.tar /tmp/tmpfile1

    echo "File Name" >> /tmp/tmpfile1

    tar -uvf /tmp/log.tar /tmp/tmpfile1

    tar -tvf /tmp/log.tar
    ...
    -rw-r--r-- root/root         0 2024-12-07 18:53 tmp/tmpfile1
    -rw-r--r-- root/root        10 2024-12-07 18:54 tmp/tmpfile1
    ```

#### `--delete` 类型

1. 您也可以使用 `--delete` 删除 `.tar` 文件内的文件。

    ```bash
    tar --delete -vf /tmp/log.tar tmp/tmpfile1

    tar --delete -vf /tmp/etc.tar etc/motd.d/
    ```

    删除时，将从归档中删除所有同名文件。

## 常用术语

一些网站提到两个术语：

* **tarfile** - 指未压缩的归档文件，如 `.tar` 文件
* **tarball** - 指压缩的归档文件，如 `.tar.gz` 和 `.tar.xz`
