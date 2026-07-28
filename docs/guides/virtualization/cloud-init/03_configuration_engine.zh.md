---
title: 3. 配置引擎
author: Wale Soyinka
contributors: Steven Spencer
tags:
  - cloud-init
  - rocky linux
  - cloud-init modules
  - automation
---

## 深入 cloud-init 模块

在上一章中，你成功启动了一个云镜像并进行了简单的定制。虽然这种方法有效，但你只有通过 `cloud-init` 的模块系统才能真正释放其强大的威力、可移植性和幂等性 (idempotency)。这些模块是 `cloud-init` 工具箱中的专用工具，旨在以声明式、可预测的方式处理特定的配置任务。

本章深入探讨模块系统，解释模块是什么、它们如何工作，以及如何使用最重要的模块来构建一台配置完善的服务器。

## 1. 配置剖析

### 什么是 cloud-init 模块

`cloud-init` 模块是一个专门的 Python 脚本，旨在处理单一的、离散的配置任务。你可以将它们视为用户管理、软件包安装和文件写入等功能的插件。

使用模块相比简单脚本（如 `runcmd`）的关键优势是 **幂等性**。幂等操作无论运行一次还是十次，都会产生相同的结果。当你声明"某用户应该存在"时，模块会确保状态得到满足——如果用户不存在就创建它，如果已存在则不执行任何操作。这使得你的配置变得可靠且可重复。

### 重温 `#cloud-config` 格式

当 `cloud-init` 看到 `#cloud-config` 头时，它将文件解释为 YAML 格式的指令集——该 YAML 文件中的顶级键直接映射到 `cloud-init` 模块。

### 模块执行与顺序

模块在启动过程的特定阶段运行，顺序由 `/etc/cloud/cloud.cfg` 定义。该流程的简化视图如下：

```
系统启动
    |
    +--- 阶段: Generator (生成器)（非常早期的启动）
    |    `--- cloud_init_modules（例如 migrator）
    |
    +--- 阶段: Local (本地)（网络之前）
    |    `---（本地设备设置模块）
    |
    +--- 阶段: Network (网络)（网络已启动）
    |    `--- cloud_config_modules（例如 users-groups、packages、write_files）
    |
    `--- 阶段: Final (最终)（后期启动）
         `--- cloud_final_modules（例如 runcmd、scripts-user）
```

顺序至关重要。例如，`users-groups` 模块在 `runcmd` 之前运行，确保脚本可以由在同一个配置中刚刚创建的用户来运行。

!!! tip "自定义 `cloud-init` 行为"

    虽然 `/etc/cloud/cloud.cfg` 定义了默认行为，但你绝对不应该直接编辑它。对于持久的、系统范围的自定义，将你自己的 `.cfg` 文件放在 `/etc/cloud/cloud.cfg.d/` 目录中。这是构建自定义镜像的标准做法，我们将在后面的章节中探讨。

## 2. 高实用性模块：日常主力

让我们使用 `virt-install` 的直接注入方法来实际操作最常用的模块。

### 模块深入：`users` 和 `groups`

妥善管理用户账户是保护新服务器实例安全的基石。`users` 模块是你实现这一目标的主要工具，它允许你创建新用户、修改现有用户、管理组成员身份，以及最重要的——注入 SSH 密钥以从首次启动就实现安全、无密码的登录。

**示例 1：创建新的管理员用户**

在此示例中，我们将配置一个名为 `sysadmin` 的专用管理员用户。通过将其添加到 `wheel` 组并提供特定的 `sudo` 规则，我们将授予该用户无密码的 `sudo` 权限。我们还将注入一个 SSH 公钥以确保安全访问。

1. **创建 `user-data.yml`：**

    ```bash
    cat <<EOF > user-data.yml
    #cloud-config
    users:
      - name: sysadmin
        groups: [ wheel ]
        sudo: [ "ALL=(ALL) NOPASSWD:ALL" ]
        shell: /bin/bash
        ssh_authorized_keys:
          - <YOUR_SSH_PUBLIC_KEY_HERE>
    EOF
    ```

2. **关键指令解释：**

    * `name`：新账户的用户名。
    * `groups`：要将用户添加到的组的列表。在 Rocky Linux 上，`wheel` 组成员身份通常用于授予管理权限。
    * `sudo`：要应用的 `sudoers` 规则列表。规则 `ALL=(ALL) NOPASSWD:ALL` 授予用户无需密码提示即可使用 `sudo` 运行任何命令的能力。
    * `ssh_authorized_keys`：要添加到用户 `~/.ssh/authorized_keys` 文件的 SSH 公钥列表。

3. **启动并验证：** 使用此 `user-data` 启动虚拟机。你应该能够以 `sysadmin` 身份通过 SSH 登录并运行 `sudo` 命令。

**示例 2：修改默认用户**

一个更常见的任务是保护云镜像提供的默认用户 (`rocky`)。在这里，我们将修改此用户以添加我们的 SSH 密钥。

1. **创建 `user-data.yml`：**

    ```bash
    cat <<EOF > user-data.yml
    #cloud-config
    users:
      - default
      - name: rocky
        ssh_authorized_keys:
          - <YOUR_SSH_PUBLIC_KEY_HERE>
    EOF
    ```

2. **关键指令解释：**

    * `default`：这个特殊条目告诉 `cloud-init` 先执行其默认用户设置。
    * `name: rocky`：通过指定现有用户的名称，模块将修改该用户而不是创建新用户。在这里，它将提供的 SSH 密钥合并到 `rocky` 用户账户中。

3. **启动并验证：** 启动虚拟机。现在你可以作为 `rocky` 用户无密码通过 SSH 登录了。

### 模块深入：`packages`

`packages` 模块提供了一种声明式的方式来管理实例上的软件，确保在启动时安装特定的应用程序。

在此示例中，我们将确保安装两个实用工具：`nginx`（高性能 Web 服务器）和 `htop`（交互式进程查看器）。我们还将指示 `cloud-init` 首先更新软件包仓库元数据，确保能找到最新版本。

1. **创建 `user-data.yml`：**

    ```bash
    cat <<EOF > user-data.yml
    #cloud-config
    package_update: true
    packages:
      - nginx
      - htop
    EOF
    ```

2. **关键指令解释：**

    * `package_update: true`：指示包管理器刷新其本地元数据。在 Rocky Linux 上，这相当于运行 `dnf check-update`。
    * `packages`：要安装的软件包名称列表。

3. **启动并验证：** 启动后，通过 SSH 登录并使用 `rpm -q nginx` 检查 `nginx` 的安装情况。

!!! note "幂等性实战"

    如果你使用相同的 `user-data` 重启此虚拟机，`packages` 模块会发现 `nginx` 和 `htop` 已经安装，不会采取进一步操作。它确保了期望的状态（软件包已存在）而不采取不必要的操作。这就是幂等性。

### 模块深入：`write_files`

这个模块功能非常强大，允许你将任何文本内容写入系统上的任何文件。它是部署应用程序配置文件、填充 Web 内容或创建辅助脚本的完美工具。

为了展示其威力，我们将使用 `write_files` 为 `nginx` Web 服务器创建一个自定义首页，同时我们在同一次运行中还安装了 nginx。

1. **创建 `user-data.yml`：**

    ```bash
    cat <<EOF > user-data.yml
    #cloud-config
    packages: [nginx]
    write_files:
      - path: /usr/share/nginx/html/index.html
        content: '<h1>Hello from cloud-init!</h1>'
        owner: nginx:nginx
        permissions: '0644'
    runcmd:
      - [ systemctl, enable, --now, nginx ]
    EOF
    ```

2. **关键指令解释：**

    * `path`：文件系统上将写入的绝对路径。
    * `content`：要写入文件的文本内容。
    * `owner`：指定应拥有该文件的用户和组（例如 `nginx:nginx`）。
    * `permissions`：以八进制格式表示的文件权限（例如 `0644`）。

3. **启动并验证：** 启动后，通过 SSH 登录并使用 `curl localhost` 查看新的首页。

!!! tip "写入二进制文件"

    `write_files` 模块不仅限于文本。通过指定 `encoding`，你可以部署二进制文件。例如，你可以使用 `encoding: b64` 来写入 base64 编码的数据。关于高级用法，请参考[官方 `write_files` 文档](https://cloudinit.readthedocs.io/en/latest/topics/modules.html#write-files)。

## 下一步

你现在已经掌握了三个最基本的 `cloud-init` 模块。通过组合它们，你可以执行大量自动化服务器配置。在下一章中，我们将处理更高级的场景，包括网络配置以及在单次运行中组合不同的 `user-data` 格式。
