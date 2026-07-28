---
title: 在 Podman 上运行 Nextcloud
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.6, 9.0
tags:
  - containers
  - nextcloud
  - podman
---

# 使用 Podman 运行 Nextcloud

## 前提条件

* 一台运行 Rocky Linux 的服务器
* 能够通过命令行文本编辑器进行编辑（此处使用 `vi`，可替换为您喜欢的编辑器）
* 能够在命令行中等程度舒适地执行操作
* 了解并接受以下事实：如果您的服务对外暴露在互联网上，存在安全风险

## 引言

许多企业、组织和个人用户已经转向使用 Nextcloud 提供的文件共享服务，并且从中获益。无论是使用它来共享个人文件、安装日历、联系人、电话应用等信息，Nextcloud 都能满足您的需求。

Nextcloud 的功能已远远超出简单文件共享的范畴。有许多可用的应用可以为它增加功能。

目前有几种可行的 Nextcloud 安装方式。您可以将其部署为一组 RPM 包，安装在实体硬件或虚拟机上；也可以使用 Docker 镜像或各种 Snaps 格式来安装。无论何种方式，只要 Nextcloud 能够正常运行即可。本文将介绍如何使用 Podman 运行 `nextcloud:p od`。

## 安装 Podman

此处使用默认的 `container-tools` 模块，它与 Rocky Linux 8 和 9 分别对应 3.0 和 4.0 版本。执行以下命令：

```bash
dnf module install container-tools
```

我们还需要安装 `podman-docker`，因为在后续过程中，我们可能会重复使用 `docker` 命令。`podman-docker` 将 `docker` 命令替换为指向 Podman 的别名。直接安装 `podman-docker`：

```bash
dnf install podman-docker
```

## 安装 `httpd-tools`

使用 `httpd-tools` 安装：

```bash
dnpm install httpd-tools
```

此步骤将安装 `htpasswd`，我们将用它来生成 Nextcloud 实例的身份验证凭据。

## 配置环境

### 创建用户凭据

!!! warning "请勿使用特殊字符"

    当您输入密码时，避免使用 `$` 或其他可能被 shell 解释的特殊字符。`htpasswd` 命令会原样接受密码，但在尝试访问 Nextcloud Web 界面时，这些特殊字符可能会干扰密码验证。请尽量使用安全的字母数字混合密码。

创建一个 "admin" 用户并设置密码。将您自己选定的用户名和密码替换进来：

```bash
htpasswd -c ./auth admin
```

添加其他用户时，省略 `-c` 选项，因为它会重新创建文件：

```bash
htpasswd ./auth user
```

此文件 (`./auth`) 将映射到容器，用于进行身份验证。

### 创建容器

使用以下脚本创建一个容器作为 Linux Pod。或许在容器操作过程中，它看起来是在做一些奇怪的事情，但实际上并没有。这个过程会比较顺利。创建 Pod：

```bash
podman pod create --hostname nextcloud --name nextcloud -p 80:80,443:443,8080:8080 --publish 8443:443
```

输出中会显示一个长容器 ID，由许多数字和字母组成。接下来，创建联合容器，它运行 Nextcloud 和 MariaDB，共享相同的命名空间(namespace)。

容器文档的书写方式目前使用的是 `docker` 命令。在我们的例子中，由于前面安装了 `podman-docker`，因此大多数 `docker` 命令都可以直接使用 `podman` 的名称。因此，我们可以轻松地参考并使用这些文档。假设您正在使用带有 `podman-docker` 别名的主机，以下命令将正常工作。

```bash
docker run -d --name mariadb --pod nextcloud -e MYSQL_ROOT_PASSWORD=password mariadb --transaction-isolation=READ-COMMITTED --binlog-format=ROW
```

上述命令将安装 MariaDB 容器，并允许 Nextcloud 使用它。

```bash
docker run -d --name nextcloud --pod nextcloud -e MYSQL_PASSWORD=password nextcloud
```

该命令安装 nextcloud 容器。同样，请注意我们在 Docker 命令上使用的是 Podman 别名。现在运行 `podman ps` 应该会返回类似以下内容：

```text
CONTAINER ID  IMAGE                               COMMAND               CREATED        STATUS            PORTS                                                                                 NAMES
927639452011  docker.io/library/mariadb:latest    --transaction-isol...  8 seconds ago  Up 8 seconds ago  0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:8080->8080/tcp, 0.0.0.0:8443->443/tcp  mariadb
ab921d0cd699  docker.io/library/nextcloud:latest  apache2-foregroun...  5 seconds ago  Up 5 seconds ago  0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:8080->8080/tcp, 0.0.0.0:8443->443/tcp  nextcloud
```

现在，您应该能够通过输入以下 URL 连接到我们的 Nextcloud 实例：

```text
http://your-domain-or-ip:8080
```

### 端口转发

如果 Nextcloud 容器运行在您只能通过 SSH 访问的离线机器上，您可能需要转发其端口。可以通过以下命令实现：

```bash
ssh -L 8080:127.0.0.1:8080 username@hostname
```

然后，在您的本地机器上，打开浏览器并导航到 `http://127.0.0.1:8080`。

## 创建 Nextcloud 管理员并完成安装

当您打开 `http://your-domain-or-ip:8080` 页面时，会要求您创建一个管理员用户。选择一个您能记住的用户名和密码。其他要求包括：

* 数据目录位置默认为 `/var/www/html/data`
* 数据库用户为 "root"
* 数据库密码为 "password"，这是在容器 `run` 命令中设置的
* 数据库名称为 "nextcloud"
* 数据库主机名必须为 "127.0.0.1"，而不是 "localhost"，因为 Nextcloud 可能尝试使用套接字文件连接到数据库

现在点击 "完成设置(Finish Setup)" 按钮，安装即告完成。接下来，您将进入 Nextcloud 仪表板：

![Nextcloud Dashboard](../images/nextcloud_dashboard.png)

## 结论

Nextcloud 可以通过多种配置进行安装，Podman 方法只是其中之一。利用 Nextcloud 的强大功能，您可以在云端或个人服务器上运行自己的文件共享服务。
