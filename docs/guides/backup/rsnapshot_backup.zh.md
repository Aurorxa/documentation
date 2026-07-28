---
title: 备份解决方案 - rsnapshot
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0, 10.0
tags:
  - backup
  - rsnapshot
---

## 前置条件

- 了解如何从命令行安装额外的仓库和快照
- 了解如何挂载机器外部的文件系统（外部驱动器、远程文件系统等）
- 知道如何使用编辑器（此处使用 `vi`，但您可以使用自己喜欢的编辑器）
- 了解一些 Bash 脚本编写
- 知道如何更改 root 用户的 `crontab`
- 了解 SSH 公钥和私钥（仅在需要从另一台服务器运行远程备份时需要）

## 简介

`rsnapshot` 是一个功能强大的备份工具，可以在任何基于 Linux 的机器上安装。它可以在本地备份单台机器，也可以从一台机器备份多台机器，例如服务器。

`rsnapshot` 完全用 Perl 编写，使用 `rsync`，没有库依赖项。安装方面没有特殊要求。在 Rocky Linux 中，您可以通过 EPEL（Enterprise Linux 额外软件包）仓库安装 `rnapshot`。如果您更倾向于使用源码方式安装，也提供了该选项。

本文档仅涵盖在 Rocky Linux 上安装 `rsnapshot`。

=== "EPEL 安装"

    ## 安装 `rsnapshot`

    除非另有说明，此处的所有命令均从命令行调用。

    ### 安装 EPEL 仓库

    您需要来自 Fedora 的 EPEL 软件仓库。使用以下命令安装该仓库：

    ```
    sudo dnf install epel-release -y
    ```

    该仓库现在将处于激活状态。

    ### 安装 `rsnapshot` 软件包

    安装 `rsnapshot` 和其他一些需要的工具：

    ``` 
    sudo dnf install rsnapshot openssh-server rsync
    ```

    如果有任何缺少的依赖项，这些会显示出来，您只需回答提示继续即可。例如：

    ```
    dnf install rsnapshot rsync
    Last metadata expiration check: 2:03:40 ago on Fri 19 Sep 2025 03:54:16 PM UTC.
    Package rsync-3.4.1-2.el10.x86_64 is already installed.
    Dependencies resolved.
    ==============================================================================================================================
    Package                         Architecture            Version                             Repository                  Size
    ==============================================================================================================================
    Installing:
    rsnapshot                       noarch                  1.5.1-1.el10_0                      epel                       112 k
    Installing dependencies:
    perl-DirHandle                  noarch                  1.05-512.2.el10_0                   appstream                   12 k

    Transaction Summary
    ==============================================================================================================================
    Install  2 Packages
    
    Total download size: 124 k
    Installed size: 388 k
    Is this ok [y/N]: y
    ```

=== "源码安装"

    ## 从源码安装 `rsnapshot`

    从源码安装 `rsnapshot` 并不困难。但它有一个缺点：如果有新版本发布，需要重新从源码安装来更新版本，而 EPEL 安装方式只需简单的 `dnf upgrade` 就能保持更新。

    ### 安装 Development Tools 并下载源码

    第一步是安装 'Development Tools' 组：

    ```
    dnf groupinstall 'Development Tools'
    ```

    您还需要一些其他软件包：

    ```
    dnf install wget unzip rsync openssh-server
    ```

    接下来，从 GitHub 仓库下载源文件。有多种方式可以完成，但本例中最简单的方法可能是直接从仓库下载 ZIP 文件。

    1. 转到 https://github.com/rsnapshot/rsnapshot
    2. 点击右侧绿色的 "Code" 按钮
    ![Code](images/code.png)
    3. 右键点击 "Download ZIP" 并复制链接地址
    ![Zip](images/zip.png)
    4. 使用 `wget` 或 `curl` 下载复制的链接。示例：
    ```
    wget https://github.com/rsnapshot/rsnapshot/archive/refs/heads/master.zip
    ```
    5. 解压 `master.zip` 文件
    ```
    unzip master.zip
    ```

    ### 构建源码

    下一步是构建。当您解压 `master.zip` 文件时，会得到一个 `rsnapshot-master` 目录。进入该目录进行构建过程。请注意，您的构建将使用所有包的默认值，所以如果您想要其他配置，需要做一些调查研究。此外，这些步骤直接取自 [GitHub 安装](https://github.com/rsnapshot/rsnapshot/blob/master/INSTALL.md) 页面：

    ```bash
    cd rsnapshot-master
    ```

    运行 `autogen.sh` 脚本生成 configure 脚本：

    ```bash
    ./autogen.sh
    ```

    !!! tip

        您可能会看到几行类似以下内容的输出：

        ```bash
        fatal: not a git repository (or any of the parent directories): .git
        ```

        这些不是致命错误。

    接下来，您需要运行 `configure` 并设置配置目录：

    ```bash
    ./configure --sysconfdir=/etc
    ```

    最后，运行 `make install`：

    ```bash
    sudo make install
    ```

    在此过程中，`rsnapshot.conf` 文件将被创建为 `rsnapshot.conf.default`。将此文件复制到 `rsnapshot.conf`，然后根据系统需要编辑它。

    ```bash
    sudo cp /etc/rsnapshot.conf.default /etc/rsnapshot.conf
    ```

    以上涵盖了配置文件的复制。下面的"配置 rsnapshot"部分将涵盖此配置文件中需要的更改。

## 挂载用于备份的驱动器或文件系统

在此步骤中，我们将展示如何挂载一个驱动器（如外部 USB 驱动器）用于备份系统。此特定步骤仅在备份单台机器或服务器时才有必要，如第一个示例所示。

1. 连接 USB 驱动器。
2. 输入 `dmesg | grep sd`，它将显示要使用的驱动器。在本例中，它是 _sda1_。  
   示例：`EXT4-fs (sda1): mounting ext2 file system using the ext4 subsystem`。
3. 不幸的是（或者幸运，取决于您的观点），大多数现代 Linux 桌面操作系统会在可能时自动挂载驱动器。这意味着，根据各种因素，`rsnapshot` 可能会失去对驱动器的跟踪。您希望驱动器每次在相同位置"挂载"或使其文件可用。  
   为此，获取 `dmesg` 命令显示的驱动器信息并输入 `mount | grep sda1`，将显示：`/dev/sda1 on /media/username/8ea89e5e-9291-45c1-961d-99c346a2628a`
4. 输入 `sudo umount /dev/sda1` 卸载外部驱动器。
5. 接下来，为备份创建挂载点：`sudo mkdir /mnt/backup`
6. 将驱动器挂载到备份文件夹位置：`sudo mount /dev/sda1 /mnt/backup`
7. 再次输入 `mount | grep sda1`，您将看到：`/dev/sda1 on /mnt/backup type ext2 (rw,relatime)`
8. 接下来，在已挂载的驱动器上创建一个用于备份的目录。本例中使用名为 "storage" 的文件夹：`sudo mkdir /mnt/backup/storage`

对于单台机器，每次连接驱动器或系统重启时，您都需要重复 `umount` 和 `mount` 步骤，或者使用脚本自动化这些命令。

建议使用自动化方式。

## 配置 `rsnapshot`

这是最重要的一步。在更改配置文件时可能会犯错。`rsnapshot` 配置要求任何元素之间使用制表符分隔，配置文件顶部有此警告。

空格字符将导致整个配置以及您的备份失败。例如，在配置文件顶部附近有 `# SNAPSHOT ROOT DIRECTORY #` 部分。如果您从头开始添加此项，您需要输入 `snapshot_root`，然后按 ++tab++，再输入 `/whatever_the_path_to_the_snapshot_root_will_be/`

好消息是，`rsnapshot` 包含的默认配置只需要细微更改即可用于本地机器的备份。不过，在开始编辑之前，备份配置文件始终是个好做法：

`cp /etc/rsnapshot.conf /etc/rsnapshot.conf.bak`

## 基础机器或单服务器备份

在此情况下，`rsnapshot` 将在本地运行以备份特定机器。在本例中，配置文件分解说明确切地展示了需要进行哪些更改。

您需要使用 `vi`（或您喜欢的编辑器）打开 `/etc/rsnapshot.conf` 文件。

首先要更改的是 _snapshot_root_ 设置。默认值为：

```text
snapshot_root   /.snapshots/
```

您需要将其更改为您创建的挂载点并添加 "storage"。

```text
snapshot_root   /mnt/backup/storage/`
```

您还需要告诉备份在驱动器未挂载时不运行。为此，移除 `no_create_root` 旁边的注释（"#" 号），如下所示：

```text
no_create_root 1
```

接下来，向下转到 `# EXTERNAL PROGRAM DEPENDENCIES #` 部分，并移除此行的注释（同样是 "#" 号）：

```text
#cmd_cp         /usr/bin/cp
```

现在它显示为：

```text
cmd_cp         /usr/bin/cp
```

虽然对于此特定配置您不需要 `cmd_ssh`，但对于其他选项您会用到它，启用它也没坏处。找到如下行：

```text
#cmd_ssh        /usr/bin/ssh
```

移除 "#" 号：

```text
cmd_ssh        /usr/bin/ssh
```

接下来，您需要跳转到 `#     BACKUP LEVELS / INTERVALS         #` 部分。

早期版本的 `rsnapshot` 有 `hourly, daily, monthly, yearly`，但现在改为 `alpha, beta, gamma, delta`。这有点令人困惑。您需要做的是对不会使用的间隔添加注释。配置中 delta 已经被注释掉了。

在本例中，您除了夜间备份外不会运行其他增量备份。只需对 alpha 和 gamma 添加注释即可。完成后，您的配置文件将如下：

```text
#retain  alpha   6
retain  beta    7
#retain  gamma   4
#retain delta   3
```

跳转到 `logfile` 行，默认如下：

```text
#logfile        /var/log/rsnapshot
```

移除注释：

```text
logfile        /var/log/rsnapshot
```

最后，跳转到 `### BACKUP POINTS / SCRIPTS ###` 部分，在 `# LOCALHOST` 部分中添加任何要添加的目录，记住在元素之间使用 ++tab++ 而不是 ++space++。

现在保存您的更改（对于 `vi`，使用 ++shift+colon+"wq!"++）并退出配置文件。

### 检查配置

您需要确保在编辑配置文件时没有添加空格或其他明显错误。为此，对配置运行 `rsnapshot` 并使用 `configtest` 选项：

`rsnapshot configtest` 如果没有错误，将显示 `Syntax OK`。

养成对特定配置运行 `configtest` 的习惯。当您进入**多机器或多服务器备份**部分时，这个习惯的重要性会更加明显。

要对特定配置文件运行 `configtest`，使用 `-c` 选项指定配置：

```bash
rsnapshot -c /etc/rsnapshot.conf configtest
```

## 首次运行备份

`configtest` 验证一切正常后，是时候首次运行备份了。您可以在测试模式下先运行，以便查看备份脚本将要做什么。

同样，在这种情况下您不一定需要指定配置，但养成这个习惯是个好主意：

```bash
rsnapshot -c /etc/rsnapshot.conf -t beta
```

这将返回类似以下内容，显示实际运行备份时将发生什么：

```bash
echo 1441 > /var/run/rsnapshot.pid
mkdir -m 0755 -p /mnt/backup/storage/beta.0/
/usr/bin/rsync -a --delete --numeric-ids --relative --delete-excluded \
    /home/ /mnt/backup/storage/beta.0/localhost/
mkdir -m 0755 -p /mnt/backup/storage/beta.0/
/usr/bin/rsync -a --delete --numeric-ids --relative --delete-excluded /etc/ \
    /mnt/backup/storage/beta.0/localhost/
mkdir -m 0755 -p /mnt/backup/storage/beta.0/
/usr/bin/rsync -a --delete --numeric-ids --relative --delete-excluded \
    /usr/local/ /mnt/backup/storage/beta.0/localhost/
touch /mnt/backup/storage/beta.0/
```

当测试符合预期后，在不使用测试模式下手动运行一次：

```bash
rsnapshot -c /etc/rsnapshot.conf beta
```

备份完成后，浏览到 `/mnt/backup` 并检查它创建的目录结构。将有一个 `storage/beta.0/localhost` 目录，后跟您指定的备份目录。

### 进一步说明

每次运行备份时，它都会创建另一个 beta 增量，0-6，即 7 天的备份。最新的备份始终是 beta.0，而昨天的备份始终是 beta.1。

这些备份中每一个的大小看起来都占用了相同（或更多）的磁盘空间，但这是因为 `rsnapshot` 使用了硬链接。要从昨天的备份恢复文件，您只需从 beta.1 的目录结构中将它们复制回来。

每个备份只是前一次运行的增量备份，但是，由于使用了硬链接，每个备份目录包含文件或指向实际备份所在目录中文件的硬链接。

要恢复文件，您无需决定从哪个目录或增量恢复，只需选择要恢复的备份时间戳。这是一个很棒的系统，比许多其他备份解决方案使用的磁盘空间要少得多。

## 设置自动运行备份

测试完成并确信一切将无问题运行后，下一步是设置 root 用户的 `crontab` 以每天自动化该过程：

```bash
sudo crontab -e
```

如果您之前没有运行过此命令，当显示 `Select an editor` 行时，选择 "vim.basic" 作为编辑器或您自己偏好的编辑器。

要将备份设置为每晚 11 点自动运行，添加以下内容到 `crontab`：

```bash
## Running the backup at 11 PM
00 23 *  *  *  /usr/bin/rsnapshot -c /etc/rsnapshot.conf beta`
```

## 多机器或多服务器备份

从一台带有 RAID 阵列或大容量存储驱动器的机器上，在本地或通过互联网连接在异地执行多台机器的备份，效果很好。

如果通过互联网运行这些备份，您需要确保每个位置都有足够的带宽来进行备份。您可以使用 `rsnapshot` 将现场服务器与异地备份阵列或备份服务器同步，以提高数据冗余。

## 假设

从远程机器运行 `rsnapshot`，在本地。在远程异地运行这种精确配置也是可能的。

在这种情况下，您需要在执行备份的机器上安装 `rsnapshot`。其他假设包括：

- 您要备份到的服务器具有允许远程机器通过 SSH 连接的防火墙规则
- 您要备份的每台服务器都安装了较新版本的 `rsync`。对于 Rocky Linux 服务器，运行 `dnf install rsync` 以更新系统的 `rsync` 版本。
- 您已以 root 用户身份连接到该机器，或者已运行 `sudo -s` 切换到 root 用户

## SSH 公钥或私钥

对于将运行备份的服务器，您需要生成一个 SSH 密钥对，用于备份期间的身份验证。在我们的示例中，您将创建 RSA 密钥。

如果您已经生成了一组密钥，可以跳过此步骤。您可以通过运行 `ls -al .ssh` 并查找 `id_rsa` 和 `id_rsa.pub` 密钥对来查看。如果没有，请使用以下链接为您的机器和要访问的服务器设置密钥：

[SSH 公钥和私钥对](../security/ssh_public_private_keys.md)

## `rsnapshot` 配置

配置文件需要与为**基础机器或单服务器备份**创建的几乎相同，但您需要更改一些选项。

snapshot root 使用默认值：

```text
snapshot_root   /.snapshots/
```

注释掉此行：

```text
no_create_root 1
#no_create_root 1
```

另一个不同之处是每台机器将有自己的配置。当您习惯之后，您只需将现有配置文件之一复制到不同名称并根据要备份的任何额外机器进行更改。

现在，您需要更改配置文件并保存它。将该文件复制为第一台服务器的模板：

```bash
cp /etc/rsnapshot.conf /etc/rsnapshot_web.conf
```

更改配置文件并创建日志和 `lockfile`，使用该机器的名称：

```text
logfile /var/log/rsnapshot_web.log
lockfile        /var/run/rsnapshot_web.pid
```

接下来更改 `rsnapshot_web.conf` 以包含要备份的目录。此处唯一不同的是目标。

以下是 "web.ourdomain.com" 配置的示例：

```bash
### BACKUP POINTS / SCRIPTS ###
backup  root@web.ourourdomain.com:/etc/     web.ourourdomain.com/
backup  root@web.ourourdomain.com:/var/www/     web.ourourdomain.com/
backup  root@web.ourdomain.com:/usr/local/     web.ourdomain.com/
backup  root@web.ourdomain.com:/home/     web.ourdomain.com/
backup  root@web.ourdomain.com:/root/     web.ourdomain.com/
```

### 检查配置并运行首次备份

现在可以检查配置以确保语法正确：

```bash
rsnapshot -c /etc/rsnapshot_web.conf configtest
```

等待 `Syntax OK` 消息。如果一切正常，可以手动运行备份：

```bash
/usr/bin/rsnapshot -c /etc/rsnapshot_web.conf beta
```

假设一切正常，您可以为邮件服务器（`rsnapshot_mail.conf`）和门户服务器（`rsnapshot_portal.conf`）创建配置文件，测试它们，并进行试运行备份。

## 自动化备份

自动化多机器或多服务器版本的备份略有不同。您需要创建一个 bash 脚本按顺序调用备份。当一个完成时，下一个将启动。此脚本将类似：

```bash
vi /usr/local/sbin/backup_all
```

内容如下：

```bash
#!/bin/bash/
# script to run rsnapshot backups in succession
/usr/bin/rsnapshot -c /etc/rsnapshot_web.conf beta
/usr/bin/rsnapshot -c /etc/rsnapshot_mail.conf beta
/usr/bin/rsnapshot -c /etc/rsnapshot_portal.conf beta
```

将脚本保存到 `/usr/local/sbin` 并使脚本可执行：

```bash
chmod +x /usr/local/sbin/backup_all
```

创建 root 的 `crontab` 以运行备份脚本：

```bash
crontab -e
```

添加此行：

```bash
## Running the backup at 11 PM
00 23 *  *  *  /usr/local/sbin/backup_all
```

## 报告备份状态

为确保一切都按计划备份，您可能希望将备份日志文件发送到电子邮件。如果您使用 `rsnapshot` 运行多机器备份，每个日志文件都有自己的名称，您可以通过[使用 postfix 进行服务器进程报告](../email/postfix_reporting.md)过程将它们发送到电子邮件以供查看。

## 恢复备份

恢复一些文件或整个备份涉及将您要从带有时间戳（日期）的目录中复制回来的文件复制回机器。

## 总结和其他资源

初次正确设置 `rsnapshot` 会有些棘手，但它可以节省大量备份机器或服务器的时间。

`rsnapshot` 功能强大、速度快，并且磁盘空间使用经济。您可以通过访问 [rsnapshot github](https://github.com/rsnapshot/rsnapshot) 找到更多关于 `rsnapshot` 的信息。
