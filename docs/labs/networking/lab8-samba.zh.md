---
author: Wale Soyinka
contributors: Steven Spencer
tags:
  - lab exercise
  - samba
  - networking
---

# 实验 8: Samba 概述

!!! info

    在继续本实验之前，你需要准备好实验 5 中的文件，路径如下：`/shared/networking/lab5-nfs_and_samba_overview.md`。

    输入命令 `lab8-samba`，启动一个名为 `lab8-samba` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，你将研究如何使用 Samba 来提供从 Linux 服务器到 Windows 客户端的正常文件服务，Windows 客户端使用 SMB/CIFS 协议访问共享资源。你将熟悉 `samba` 服务器和客户端管理。

    !!! knowledge "知识要点"

        本实验将让你亲身体验如何使用 Samba。你将逐步完成以下步骤：创建公开可访问的 Samba 导出，创建受密码保护的 Samba 导出和用户，以及从客户端访问这两个导出。在你完成 Samba 设置的过程中，你还会遇到可能出错的问题，并获得排查这些问题的工具。

## 介绍

本实验旨在帮助你深入了解 Samba 服务器的操作。你将通过创建实验室拓扑来获得 Samba 的实践经验。

### 先决条件

在开始实验之前，你需要做好以下准备：

* 一个具有 Samba 服务器功能的节点，例如 `SAMBA_SERVER`
* 一个用作客户端并运行 Linux 的节点，例如 `SAMBA_CLIENT`
* 或者，你也可以使用一个真正的 Windows 客户端作为附加的 Samba 客户端，但这并非必需
* 两台机器（服务器和客户端）均安装了 Rocky Linux 9.5
* 管理员（root）或具有完整 sudo 访问权限的帐户
* 所有机器能够相互 ping 通

### Samba 服务器设置

在名为 `SAMBA_SERVER` 的节点上完成以下操作：

1. 安装必要的 `samba` 服务器软件包。在 Rocky Linux 上，这意味着 `samba` 和 `samba-common` 以及其他一些软件包。

    !!! question "问题"

        你使用什么命令来安装核心 Samba 软件包？

2. Samba 服务器守护进程为 `smbd`。你需要启用 `smbd` 服务并使其在机器重启后也能正常工作。

    !!! question "问题"

        启用和启动 `smbd` 的正确命令是什么？

3. 在系统防火墙中打开 samba 端口。

    !!! question "问题"

        你需要打开或允许通过防火墙的默认 Samba 端口是什么？

4. 现在是最棘手的部分——Samba 的配置。但实际上，它更简单，因为一切都浓缩在一个文件里！它的位置是 `/etc/samba/smb.conf`。你会发现该文件已经存在，并且可能有一些预配置的内容。

    通常，你会添加一个 `[share]` 节，如果你想发布一个名为 "public" 的公开共享。如果想让任何人都可以列出、读取和写入该共享，可以查看以下示例：

    ```bash
    [public]
        browseable = Yes
        path = /srv/smb_share
        guest ok = Yes
        read only = No
        create mask = 777
        directory mask = 777
    ```

    但这只是一个起点！以它为指导，在其基础上添加你的自定义设置。确保你创建的共享路径存在。如果需要，可以在 `/etc/samba/smb.conf` 中创建类似以下内容的共享。

    创建两个共享：

    * **一个名为 `public` 的共享**
        * 从服务器上的 `/srv/smb_share/public` 提供数据。
        * `public` 共享允许访客（即"guest ok"）访问。
        * 该共享内容应可被访问该共享的任何人写入（即"read only = no"）。
    * **第二个共享名为 `protected`**
        * 从服务器上的 `/srv/smb_share/protected` 提供数据。
        * `protected` 共享只能由有效的用户和组凭证访问。
        * 该共享内容应可被访问该共享的任何人写入（即"read only = no"）。

    !!! question "问题"

        完成 `smb.conf` 配置后，建议你将配置通过 `testparm` 运行。还需要做什么才能使配置更改生效？

    !!! tip

        有没有一些设置可以保护 Samba 服务器免受 Windows 操作系统的 SMB 协议某些老版本遗留下来的安全漏洞？

5. 在本地文件系统上创建共享目录，并在这些目录中创建一些文件，以便后续测试。

6. 对于 `protected` 共享，只有一个名为 "sambauser" 的用户才能访问。你需要在系统上创建一个具有相同名称（"sambauser"）的用户，然后使用 `smbpasswd` 为该用户设置一个仅用于 Samba 的密码。

7. 为两个共享正确设置 SELinux 上下文/布尔值。

    !!! question "问题"

        哪个布尔值使 Samba 能够将用户的主目录作为共享加载？（提示：使用 `getsebool` 和 `setsebool`）

8. 你现在应该有一个工作的 `SAMBA_SERVER`。

### Samba 客户端设置

现在，你需要作为 Samba 客户端登录到 `SAMBA_CLIENT` 计算机。

1. 要浏览基于 SMB 的共享，你需要一些客户端工具。安装 `samba-client` 软件包。

2. 要使用 `smbclient` 列出 `SAMBA_SERVER` 上可用的共享，可以运行：
    ```bash
    smbclient -L //SAMBA_SERVER -U%
    ```
    或者，你可以尝试
    ```bash
    smbclient -L //SAMBA_SERVER -U guest%
    ```

    !!! question "问题"

        分别解释这些不同命令的作用以及它们的区别：

        a. 第一个 `smbclient` 命令：
        b. 第二个 `smbclient` 命令：

3. `smbclient` 具有类似 FTP 的界面，可用于浏览和传输文件。首先尝试以 `sambauser` 用户身份访问 `protected` 共享。然后在提示符下输入 Samba 密码。
    ```bash
    smbclient //SAMBA_SERVER/protected -U sambauser
    ```

    这应该会显示一个提示符：
    ```text
    smb: \>
    ```

    !!! question "问题"

        在 `smb: \>` 提示符下，列出可用于管理文件和目录的命令。

4. 与 NFS 相比，使用 `smbclient` 对于常规使用来说有点繁琐，因此，你可以使用标准的 Linux 挂载工具来挂载 Samba `protected` 共享。这可以使用 `mount.cifs` 或 `mount -t cifs` 命令来完成。

    你的任务是依次挂载两个共享（`public` 和 `protected`）。在挂载 `protected` 共享时，确保使用 "sambauser" 用户及其凭证。

    将共享挂载到 `/mnt/` 目录下的适当位置。

    正确挂载共享后，你可以像普通的本地磁盘一样使用它们。试着将一些文件复制到挂载的共享中，或从挂载的共享中复制文件。

    !!! question "问题"

        你用来挂载 `protected` 共享的命令是什么？

    !!! knowledge "知识要点"

        你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
