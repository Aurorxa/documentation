---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - lab exercise
  - file system
  - systems administration
---

# 实验 6: 文件系统

!!! info

    输入命令 `lab6-the_file_system`，启动一个名为 `lab6-the_file_system` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，你将更深入地探索 Linux 文件系统。在之前的实验中，你已经了解了 Linux 支持的各具特色的文件系统的基本概念，如 `xfs`、`ext4`、`btrfs` 等。你还学会了基本的分区、格式化和临时挂载。

    !!! knowledge "知识要点"

        在本实验中，你将：

        * 加深对 Linux 文件系统的理解
        * 练习检查和使用不同类型的文件系统
        * 练习创建、挂载、扩展、缩小和移除各种类型的文件系统
        * 了解交换空间（swap）和文件系统的其他关键方面
        * 使用访问控制列表（ACLs）和磁盘配额（quotas）

## 目标

完成本实验后，你应该能够：

* 识别并检查各种文件系统
* 创建各种文件系统并使其可在系统上使用
* 列出并管理文件系统挂载和交换空间
* 了解磁盘配额

## 先决条件

在开始本实验之前，你需要：

* 1 台安装了 Rocky Linux 的机器
* 具有 root 访问权限或 sudo 凭据
* 至少 3 个额外的未格式化磁盘（每个大约 1-5 GB）

## 实验

### 识别和检查文件系统

1. 列出你的系统上当前可用的块设备。

    !!! question "问题"

        你能识别出每个分区的文件系统类型吗？如果可以，如何确定每个分区的文件系统类型？

2. 使用 `lsblk` 命令确保你看到了至少三个磁盘设备，用于本次练习。
3. 使用 `blkid` 查看可用设备的属性。

### 创建文件系统

在此，你将在不同设备上创建不同且各具特色的一套文件系统。

#### 创建 EXT4 文件系统

1. 在可用设备之一（例如 `/dev/sdb`）上创建一个 `ext4` 文件系统，卷标为 `scratch`。

#### 创建 XFS 文件系统

1. 在另一个可用磁盘（例如 `/dev/sdc`）上创建一个 `xfs` 文件系统，卷标为 `scratch`。

#### 创建 BTRFS 文件系统

1. 在你的第三个可用设备（例如 `/dev/sdd`）上创建一个 `btrfs` 文件系统，卷标为 `scratch`。

创建好这三个文件系统后，我们来实际使用它们。首先创建一个挂载点目录：

```bash
mkdir -p /mnt/scratch-volume /mnt/scratch2-volume /mnt/scratch3-volume
```

现在验证你是否可以看到你创建的所有三个文件系统：

```bash
lsblk -f
```

!!! question "问题"

    对于你创建的每个文件系统，需要在 `/etc/fstab` 中添加什么内容，使它们能够永久可访问？

### 扩展和缩小文件系统

虽然无法缩小 xfs 文件系统的实际大小，但你可以在某些条件下缩小卷组或其他内容。在本节中，我们将绕过这个限制，通过在设备上创建第二个文件系统来练习缩小操作。

#### 扩展文件系统

1. 首先，挂载你之前创建的 `ext4` 卷：

    ```bash
    mount /dev/sdb /mnt/scratch-volume
    ```

    检查当前大小：
    ```bash
    df -h | grep scratch
    ```

2. 检查你的设备是否有多余的空间可以添加。使用 `parted` 或 `fdisk` 调整分区大小。
3. 扩展物理卷上的文件系统本身：

    ```bash
    resize2fs /dev/sdb
    ```

    !!! question "问题"

        扩展文件系统时，必须先调整物理分区大小，然后再调整文件系统本身。这两个步骤分别对应的命令是什么？

#### 缩小文件系统

1. 缩小文件系统需要非常谨慎。对于 `ext4`，执行此操作前必须先卸载卷。如果你已经挂载了该卷，请先卸载。
2. 假设你想将文件系统大小减小到 2GB。首先检查文件系统：

    ```bash
    e2fsck -f /dev/sdb
    ```

    然后使用 `resize2fs` 缩小文件系统：

    ```bash
    resize2fs /dev/sdb 2G
    ```

    之后，使用 `parted` 或 `fdisk` 调整分区大小。

    !!! question "问题"

        为什么文件系统必须处于卸载状态才能缩小？如果强制执行会有什么风险？

3. 重新挂载文件系统并查看其新尺寸。

### 移除文件系统

移除文件系统意味着清除设备上的签名。在执行此操作之前，请确保设备未被挂载且所有重要数据已备份。

```bash
wipefs -a /dev/sdb
```

!!! question "问题"

    使用 `wipefs -a` 与使用 `dd if=/dev/zero of=/dev/sdb bs=1M count=100` 有什么区别？

### 交换空间

Linux 交换空间用于当 RAM 已被大量占用时，为系统提供额外的内存。将数据从 RAM 转移到硬盘交换空间的过程称为 "swapping"。你可以使用磁盘分区或文件作为交换空间；系统管理员通常会决定哪种方式最适合手头的任务。

要查看系统是否已使用交换空间，以及当前的使用量，请运行：

```bash
free -h
```

或

```bash
swapon --show
```

#### 创建交换空间作为分区

1. 在你之前擦除的磁盘上（例如 `/dev/sdb`）创建一个新分区。使用 `fdisk` 或 `parted`。
2. 使用 `mkswap` 将分区初始化为交换空间，然后用 `swapon` 启用。
3. 启用后，你可以再次运行 `free -h` 或 `swapon --show` 来确认。
4. 如需使更改永久生效，请在 `/etc/fstab` 中添加相应条目。
5. 练习完成后，关闭交换分区：

    ```bash
    swapoff /dev/sdb1
    ```

#### 创建交换文件

除了使用设备，你还可以使用磁盘上的文件用作交换空间。这是测试或短期使用的好选择。

1. 创建一个 1GB 大小的文件，用于交换空间：

    ```bash
    dd if=/dev/zero of=/swapfile bs=1M count=1024
    ```

2. 确保权限已加固：

    ```bash
    chmod 600 /swapfile
    ```

3. 使用 `mkswap` 准备文件，并用 `swapon` 启用。

### 其他注意事项

以下是一些你可能需要了解的与文件系统相关的额外主题，特别是你将使用一个名为 `xfs_admin` 的工具。

1. 检查 `xfs` 文件系统类型的分区标签。
2. 使用 `xfs_admin` 为 xfs 文件系统设置新标签。
3. 给出你在 fstab 中使用 `LABEL=`、`UUID=` 或 `/dev/<device>` 指定文件系统的原因。

## ACLS 与配额

随着你对文件系统的深入理解，管理权限和理解配额变得愈发重要。下面我们将探讨这两个领域。

### 访问控制列表（ACL）

在类 Unix 操作系统上，文件系统访问控制列表是指定特定用户或组对单个文件或目录的权限的列表。

1. 首先，确保你的文件系统支持 `ACL`。对于 `ext4` 和 `xfs`，`ACL` 通常默认已挂载并已开启。
2. 创建一个测试文件并查看其默认权限：

    ```bash
    touch /mnt/scratch-volume/acltest
    getfacl /mnt/scratch-volume/acltest
    ```

3. 为某个用户设置扩展 ACL：

    ```bash
    setfacl -m u:unreasonable:rwx /mnt/scratch-volume/acltest
    getfacl /mnt/scratch-volume/acltest
    ```

    这会赋予用户 `unreasonable` 对该文件的 `rwx` 权限，但不改变标准用户和组的所有权。

4. 在授予权限后，尝试以 `unreasonable` 用户身份访问该文件——应该可以正常工作。
5. 为某个组设置 ACL，然后尝试以该组成员身份访问该文件。
6. 同样尝试设置默认 ACL，使它们能被新创建的文件和目录继承。

    !!! question "问题"

        ACL 在系统配置和应用程序部署中有哪些实际用途？

### 磁盘配额

配额用于限制每个用户或组的磁盘空间使用量，防止少数用户消耗过多的存储资源。

配置配额有几个基本步骤。它们包括：

* 安装配额工具
* 挂载文件系统并开启配额支持
* 运行 quotacheck
* 打开配额
* 配置用户的配额
* 生成报告

1. 确保已安装 `quota` 软件包。
2. 选择或创建一个文件系统，用于实验配额。这里我们将使用 `/mnt/2gb-scratch2-volume`。
3. 在文件系统上启用配额。对于 `xfs`，在挂载选项中包含以下内容：`uquota` 或 `usrquota`；对于用户和组配额，也可以指定 `gquota` 或 `grpquota`。
4. 挂载文件系统后，设置一个名为 `unreasonable` 的测试用户：

    ```bash
    useradd -m unreasonable
    ```

    为该用户在此卷上开启配额：

    ```bash
    xfs_quota -x -c 'limit bsoft=500m bhard=1000m unreasonable' /mnt/2gb-scratch2-volume
    ```

5. 现在，以 `unreasonable` 用户的身份在该卷上创建一个文件：

    ```bash
    su - unreasonable
    fallocate -l 1.5G /mnt/2gb-scratch2-volume/LARGE-USELESS-FILE.tar
    ```

6. 结果令 `unreasonable` 用户感到沮丧。以 root 身份运行 `quota` 命令，查看文件是否生效。运行报告命令查看是否超出限制：

    ```bash
    xfs_quota -x -c report /mnt/2gb-scratch2-volume
    ```

    输出类似于：

    ```text
    User quota on /mnt/2gb-scratch2-volume (/dev/sdc1)
                                   Blocks
    User ID          Used       Soft       Hard    Warn/Grace
    ---------- --------------------------------------------------
    root                0          0          0     00 [--------]
    unreasonable     1536000   512000   1024000     00 [7 days]
    ```

    !!! question "问题"

        从上方输出中 `unreasonable` 用户的 `grace` 列来看，该用户还有多少宽限期？

7. 根据报告，你注意到 `unreasonable` 用户已超出配额限制。你找到了违规文件，并帮助 `unreasonable` 用户"清理"了它，使之重新合规。输入：

    ```bash
    rm -rf /mnt/2gb-scratch2-volume/LARGE-USELESS-FILE.tar
    ```

8. 使用 `su` 命令临时以 `unreasonable` 用户身份登录，并尝试以该用户身份创建额外的文件或目录。输入：

    ```bash
    su - unreasonable
    ```

9. 以 `unreasonable` 用户身份登录后，你注意到之前创建的 `/mnt/2gb-scratch2-volume/LARGE-USELESS-FILE.tar` 文件不见了！你感到恼火，决定再次创建它。输入：

    ```bash
    dd if=/dev/zero  of=/mnt/2gb-scratch2-volume/LARGE-USELESS-FILE.tar bs=10240
    ```

    **输出**

    ```bash
    ...<SNIP>...
    dd: error writing '/mnt/2gb-scratch2-volume/LARGE-USELESS-FILE.tar': Disk quota exceeded
    10001+0 records in
    10000+0 records out
    102400000 bytes (102 MB, 98 MiB) copied, 0.19433 s, 527 MB/s
    ```

    嗯……有意思，你嘀咕着。

10. 尝试在 `/mnt/2gb-scratch2-volume/` 下创建一个名为 `test` 的目录。一个空目录不应该占用太多磁盘空间，于是你输入：

    ```bash
    mkdir /mnt/2gb-scratch2-volume/test
    ```
    ```text
    mkdir: cannot create directory '/mnt/2gb-scratch2-volume/test': Disk quota exceeded
    ```

11. 查看 LARGE-USELESS-FILE.tar 文件的大小。输入：

    ```bash
    ls -l /mnt/2gb-scratch2-volume/LARGE-USELESS-FILE.tar
    ```
    ```text
    -rw-rw-r-- 1 unreasonable unreasonable 102400000 Oct  5 19:37 /mnt/2gb-scratch2-volume/LARGE-USELESS-FILE.tar
    ```

    !!! question "问题"

        发生了什么？

12. 带着无知和沮丧，`unreasonable` 用户输入：

    ```bash
    man quota
    ```

    !!! note

        `unreasonable` 用户必须对他创建的 `LARGE-USELESS-FILE.tar` 采取行动。除非将总文件大小控制在限制以内，否则他什么都做不了。

13. 关于 Linux 文件系统的实验到此结束。
