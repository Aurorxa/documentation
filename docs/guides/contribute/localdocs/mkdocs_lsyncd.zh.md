---
title: Incus 方法
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.5, 8.6
tags:
  - contribute
  - local environment LXD
  - local environment incus
---

!!! info

    虽然此方法仍然适用于 LXD，但作者更偏好使用 Incus。原因是从开发角度来看，Incus 似乎领先于 LXD，包括可用的镜像。截至 2025 年 9 月，Incus 上有 Rocky Linux 10 和其他 RHEL 重编译版 10 的镜像。LXD 镜像目前仅包含九个构建版本。这可能与 2023 年 12 月 Linux Containers 项目负责人 Stéphane Graber 宣布的[许可证变更](https://stgraber.org/2023/12/12/lxd-now-re-licensed-and-under-a-cla/)有关。

    此外，此流程仍然适用于当前的文档版本管理。如果在任何版本分支（main、rocky-9 和 rocky-8）上创建或编辑文档，同步到容器的文档将显示正确的内容。这意味着您可以继续按原样使用此流程。我已添加了关于版本管理的额外说明。

!!! tip

    如果您使用 Rocky Linux 10 作为工作站，请注意，在本文档重写时，`lsyncd` 在 EPEL 中不可用。您需要使用从源码安装的方法。

## 引言

有多种方法可以运行 `mkdocs` 的副本，以确切了解您的 Rocky Linux 文档在合并到实时系统时将如何显示。本文专门讨论在本地工作站上使用 `incus` 容器，将 `mkdocs` 中的 Python 代码与您可能正在处理的其他项目分开。

建议将项目分开，以避免对工作站的代码造成问题。

## 前提条件和假设

- 熟悉并适应命令行操作
- 能够轻松使用编辑、SSH 和同步工具，或愿意跟随学习
- Incus 参考 - 有关于[在服务器上构建和使用 `incus` 的详细文档](../../../books/incus_server/00-toc.md)，但您只需在 Linux 工作站上进行基本安装
- 使用 `lsyncd` 进行文件镜像。请参阅[相关文档](../../backup/mirroring_lsyncd.md)
- 您需要为本地工作站上的用户和 "root" 用户生成公钥(public key)，使用[此文档](../../security/ssh_public_private_keys.md)
- 我们的桥接接口运行在 10.56.233.1 上，容器运行在 10.56.233.189 上。但您的网桥和容器的 IP 会有所不同。
- 本文档中的 "youruser" 代表您的用户 ID
- 假设您已经在工作站上使用文档仓库的克隆进行文档开发

## `mkdocs` 容器

### 创建容器

第一步是创建 `incus` 容器。使用容器的默认设置（桥接接口）在这里完全没问题。

您将向工作站添加一个用于 `mkdocs` 的 Rocky 容器。只需将其命名为 "mkdocs"：

```bash
incus launch images:rockylinux/10 mkdocs
```

该容器需要是代理(proxy)。默认情况下，当 `mkdocs serve` 启动时，它运行在 127.0.0.1:8000 上。当它在本地工作站上而没有容器时，这是可以的。但是，当它在本地工作站上的 `incus` **容器**中时，您需要为容器设置代理端口。使用以下命令：

```bash
incus config device add mkdocs mkdocsport proxy listen=tcp:0.0.0.0:8000 connect=tcp:127.0.0.1:8000
```

在上面的行中，"mkdocs" 是我们的容器名，"mkdocsport" 是您为代理端口给定的任意名称，类型为 "proxy"，您正在监听所有 TCP 接口上的 8000 端口，并连接到 localhost 上的 8000 端口。

!!! Note

    如果您在网络中的另一台机器上运行 `incus` 实例，请记住确保防火墙中打开了 8000 端口。

### 安装软件包

首先，使用以下命令进入容器：

```bash
incus shell mkdocs bash
```

对于 Rocky Linux 10，您需要一些软件包：

```bash
dnf install git openssh-server python3-pip rsync
```

安装完成后，需要启用并启动 `sshd`：

```bash
systemctl enable --now sshd
```

### 容器用户

您需要为 root 用户设置密码，然后将您的用户（您在本地机器上使用的用户）添加到 sudoers 列表中。您当前是 "root" 用户。要更改密码，请输入：

```text
passwd
```

设置一个安全且易记的密码。

接下来，添加您的用户并设置密码：

```bash
adduser youruser
passwd youruser
```

将您的用户添加到 sudoers 组：

```bash
usermod -aG wheel youruser
```

您应该能够从工作站以 root 或您的用户身份 SSH 到容器并输入密码。在继续之前确保可以做到这一点。

## 为 root 和您的用户配置 SSH

在此流程中，root 用户（至少）需要能够在无需输入密码的情况下 SSH 到容器中。这是因为您将实现的 `lsyncd` 进程。这里假设您可以在本地工作站上通过 `sudo` 切换到 root 用户：

```bash
sudo -s
```

同时假设 root 用户在 `./ssh` 目录中有 `id_rsa.pub` 密钥。如果没有，请使用[此流程](../../security/ssh_public_private_keys.md)生成一个：

```bash
ls -al .ssh/
drwx------  2 root root 4096 Feb 25 08:06 .
drwx------ 14 root root 4096 Feb 25 08:10 ..
-rw-------  1 root root 2610 Feb 14  2021 id_rsa
-rw-r--r--  1 root root  572 Feb 14  2021 id_rsa.pub
-rw-r--r--  1 root root  222 Feb 25 08:06 known_hosts
```

要在不输入密码的情况下获取对容器的 SSH 访问权限，如果 `id_rsa.pub` 密钥存在，只需运行：

```bash
ssh-copy-id root@10.56.233.189
```

但是，对于您的用户，您需要将整个 `.ssh/` 目录复制到您的容器中。您将为此用户保持所有内容不变，以便您对 GitHub 的 SSH 访问保持不变。

要将所有内容复制到容器中，您需要以您的用户身份执行此操作，**而非** `sudo`：

```bash
scp -r .ssh/ youruser@10.56.233.189:/home/youruser/
```

接下来，以您的用户身份 SSH 到容器中：

```bash
ssh -l youruser 10.56.233.189
```

请确保一切相同。您将使用 `ssh-add` 执行此操作。您还必须确保有 `ssh-agent` 可用：

```bash
eval "$(ssh-agent)"
ssh-add
```

## 克隆仓库

您需要克隆两个仓库，但无需添加任何 `git` 远程。此处的文档仓库将仅显示当前文档（从您的工作站镜像而来）和文档站点。

rockylinux.org 仓库用于运行 `mkdocs serve`，并将使用镜像作为其源。以非 root 用户身份运行所有这些步骤。如果无法以您的用户 ID 克隆仓库，那么在使用 `git` 时您的身份**确实**存在问题，您需要回顾上面关于重新创建密钥环境（上文）的最后几个步骤。

首先，克隆文档库：

```bash
git clone git@github.com:rocky-linux/documentation.git
```

接下来，克隆 docs.rockylinux.org：

```bash
git clone git@github.com:rocky-linux/docs.rockylinux.org.git
```

如果出现错误，请返回到前面的步骤，在继续之前确保所有步骤都正确。

## 设置 `mkdocs`

使用 docs.rockylinux.org 目录中的 "requirements.txt" 文件通过 `pip3` 安装所需的插件。虽然此过程会警告您使用 root 用户向系统目录写入更改，但您仍然必须以 root 用户身份运行它。

您在此处使用 `sudo`。

进入目录：

```bash
cd docs.rockylinux.org
```

然后运行：

```bash
sudo pip3 install -r requirements.txt
```

接下来，您必须为 `mkdocs` 设置一个额外的目录。`mkdocs` 需要创建一个 docs 目录，然后将 `documentation/docs` 目录链接在其下方。使用以下命令：

```bash
mkdir docs
cd docs
ln -s ../../documentation/docs
```

### 测试 `mkdocs`

现在 `mkdocs` 已经设置好了，尝试启动服务器。请记住，此过程会提示它似乎是生产环境。它不是，所以忽略警告。使用以下命令启动 `mkdocs serve`：

```bash
mkdocs serve -a 0.0.0.0:8000
```

您将在控制台中看到类似以下内容：

```bash
INFO     -  Building documentation...
WARNING  -  Config value: 'dev_addr'. Warning: The use of the IP address '0.0.0.0' suggests a production environment or the use of a
            proxy to connect to the MkDocs server. However, the MkDocs' server is intended for local development purposes only. Please
            use a third party production-ready server instead.
INFO     -  Adding 'sv' to the 'plugins.search.lang' option
INFO     -  Adding 'it' to the 'plugins.search.lang' option
INFO     -  Adding 'es' to the 'plugins.search.lang' option
INFO     -  Adding 'ja' to the 'plugins.search.lang' option
INFO     -  Adding 'fr' to the 'plugins.search.lang' option
INFO     -  Adding 'pt' to the 'plugins.search.lang' option
WARNING  -  Language 'zh' is not supported by lunr.js, not setting it in the 'plugins.search.lang' option
INFO     -  Adding 'de' to the 'plugins.search.lang' option
INFO     -  Building en documentation
INFO     -  Building de documentation
INFO     -  Building fr documentation
INFO     -  Building es documentation
INFO     -  Building it documentation
INFO     -  Building ja documentation
INFO     -  Building zh documentation
INFO     -  Building sv documentation
INFO     -  Building pt documentation
INFO     -  [14:12:56] Reloading browsers
```

如果一切操作正确，您应该能够打开网页浏览器并转到容器 IP 地址的 :8000 端口，查看文档站点。

在我们的例子中，在浏览器地址栏输入以下内容（**注意**：为避免 URL 断开，此处的 IP 为 "your-server-ip"。您需要替换为实际 IP）：

```bash
http://your-server-ip:8000
```

## `lsyncd`

如果您在网页浏览器中看到了文档，就快完成了。最后一步是保持容器中的文档与本地工作站上的文档同步。

如前所述，您在此处使用 `lsyncd` 来完成。

`lsyncd` 的安装取决于您的 Linux 版本。[此文档](../../backup/mirroring_lsyncd.md)涵盖了在 Rocky Linux 上从 EPEL (Extra Packages for Enterprise Linux) 和源码安装的方法。如果您使用的是其他 Linux 发行版（如 Ubuntu），它们通常有自己的软件包，但有些细微差别。

例如，Ubuntu 对配置文件的命名方式不同。只需注意，如果您使用的是 Rocky Linux 以外的其他 Linux 工作站，并且不想从源码安装，您的平台上可能有可用的软件包。

现在，我们假设您使用的是 Rocky Linux 工作站以及包含文档中描述的 RPM 安装方法。

!!! note

    截至本文撰写时，`lsyncd` 在 Rocky Linux 10 的 EPEL 中不可用。如果那是您的工作站版本，您需要使用源码安装方法。

### 配置

!!! Note

    守护进程(daemon)必须以 root 用户运行，因此您必须是 root 才能创建配置文件和日志。对此，我们假设使用 `sudo -s`。

您需要有可用于写入的 `lsyncd` 日志：

```bash
touch /var/log/lsyncd-status.log
touch /var/log/lsyncd.log
```

您还需要创建一个排除文件，即使在这种情况下您不排除任何内容：

```bash
touch /etc/lsyncd.exclude
```

最后，您需要创建配置文件。在本例中，我们使用 `vi` 作为编辑器，但请使用您觉得舒适的任何编辑器：

```bash
vi /etc/lsyncd.conf
```

然后将此内容放入该文件并保存。请务必将 "youruser" 替换为您的实际用户名，并将 IP 地址替换为您自己的容器 IP：

```bash
settings {
   logfile = "/var/log/lsyncd.log",
   statusFile = "/var/log/lsyncd-status.log",
   statusInterval = 20,
   maxProcesses = 1
   }

sync {
   default.rsyncssh,
   source="/home/youruser/documentation",
   host="root@10.56.233.189",
   excludeFrom="/etc/lsyncd.exclude",
   targetdir="/home/youruser/documentation",
   rsync = {
     archive = true,
     compress = false,
     whole_file = false
   },
   ssh = {
     port = 22
   }
}
```

假设您在安装时已启用了 `lsyncd`，此时需要启动或重启该进程：

```bash
systemctl restart lsyncd
```

为确保一切正常工作，请检查日志，特别是 `lsyncd.log`，如果一切正常启动，应显示类似以下内容：

```bash
Fri Feb 25 08:10:16 2022 Normal: --- Startup, daemonizing ---
Fri Feb 25 08:10:16 2022 Normal: recursive startup rsync: /home/youruser/documentation/ -> root@10.56.233.189:/home/youruser/documentation/
Fri Feb 25 08:10:41 2022 Normal: Startup of "/home/youruser/documentation/" finished: 0
Fri Feb 25 08:15:14 2022 Normal: Calling rsync with filter-list of new/modified files/dirs
```

## 版本管理说明

您需要一份 [Rocky Linux 文档仓库](https://github.com/rocky-linux/documentation)的克隆。这部分很重要，因为如果您克隆的是自己的 fork 仓库，那么您执行 `git checkout` 切换到 `rocky-8` 和 `rocky-9` 分支的能力将不存在。只有 `main` 分支可用。

### GitHub 工作站设置

这些步骤不是为您的容器准备的，而是为您工作站上的文档副本准备的：

1. 克隆 Rocky Linux 文档仓库：

    ```bash
    git clone git@github.com:rocky-linux/documentation.git
    ```

2. `git remote` 名称将是 "upstream" 而不是 "origin"。使用以下命令检查远程名称：

    ```bash
    git remote -v
    ```

    克隆后立即显示：

    ```bash
    origin git@github.com:rocky-linux/documentation.git (fetch)
    origin git@github.com:rocky-linux/documentation.git (push)
    ```

    使用以下命令重命名远程：

    ```bash
    git remote rename origin upstream
    ```

    再次运行 `git remote -v`，您将看到：

    ```bash
    upstream git@github.com:rocky-linux/documentation.git (fetch)
    upstream git@github.com:rocky-linux/documentation.git (push)
    ```

3. 将您的 fork 添加为名为 "origin" 的远程。替换您的实际 GitHub 用户名：

    ```bash
    git remote add origin git@github.com:[your-github-user-name]/documentation.git
    ```

    再次运行 `git remote -v`，您将看到：

    ```bash
    origin git@github.com:[your-github-user-name]/documentation.git (fetch)
    origin git@github.com:[your-github-user-name]/documentation.git (push)
    upstream git@github.com:rocky-linux/documentation.git (fetch)
    upstream git@github.com:rocky-linux/documentation.git (push)
    ```

4. 您需要用版本分支（除 `main` 之外）填充您的 fork。`main` 分支当前包含版本 10 的信息。您希望将 `rocky-8` 和 `rocky-9` 分支合并到您的 fork 中，以便编辑这些较旧版本的文档。第一步是 `git checkout` 这些分支名称：

    ```bash
    git checkout rocky-8
    ```

    首次执行时，您将看到：

    ```bash
    branch 'rocky-8' set up to track 'upstream/rocky-8'.
    Switched to a new branch 'rocky-8'
    ```

    接下来，将分支推送到您的 fork：

    ```bash
    git push origin rocky-8
    ```

    这会显示为正在创建新的 pull request，但当您检查 fork 分支内容时，您将看到 `rocky-8` 现在是其中一个分支。

    对 `rocky-9` 分支重复这些步骤。

### 这如何适用于此流程

在创建分支后，如果您只想编辑 `rocky-9` 的 `README.md`，您需要基于 `rocky-9` 版本分支创建一个新分支：

```bash
git checkout -b fixes_for_rocky9_readme rocky-9
```

然后正常编辑文档。在保存工作时，您的容器文档将更新，运行本文档中描述的 `mkdocs serve` 将显示该内容。

完成后并将更改推送到您的 fork 以创建 pull request，您可以再次检回 `main` 分支。由于您的所有工作都在检出的 rocky-9 分支中，容器中同步的文档将恢复到您开始该流程之前的状态。通过这种方式，您始终可以跟踪您的工作，无论您正在处理哪个版本。您的容器将保持与本地工作站上的内容同步。

## 结论

您可以在工作站上处理文档，同时在容器的同步副本中查看更改。推荐的做法是所有 Python 代码必须与您可能正在开发的任何其他 Python 代码分开运行。使用 `incus` 容器使这变得更加容易。
