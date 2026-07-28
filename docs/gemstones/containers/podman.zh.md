---
title: Podman
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova, Christian Steinert
date: 2024-03-07
tags:
  - docker
  - podman
---

# 简介

[Podman](https://podman.io/) 是一个兼容 Docker 的替代容器运行时，与 Docker 不同的是，它包含在 Rocky Linux 仓库中，并且可以作为 `systemd` 服务运行容器。

## 安装 Podman

使用 `dnf` 工具安装 Podman：

```bash
dnf install podman
```

## 添加容器

我们以运行一个 [Nextcloud](https://nextcloud.com/) 自托管云平台为例：

```bash
podman run -d -p 8080:80 nextcloud
```

您将收到提示，要求选择要从中下载的容器注册表。在我们的示例中，我们将使用 `docker.io/library/nextcloud:latest`。

下载完 Nextcloud 容器后，它将开始运行。

在 Web 浏览器中输入 **ip_address:8080**（假设您已在 `firewalld` 中打开了该端口），然后设置 Nextcloud：

![Nextcloud in container](../images/podman_nextcloud.png)

## 作为 `systemd` 服务运行容器

### 使用 `quadlet`

从 4.4 版本开始，Podman 附带 [Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)，这是一个 systemd 生成器，可以为无 root 和有 root 的 systemd 服务生成单元文件。

有 root 服务的 Quadlet 文件可以放置在：

- `/etc/containers/systemd/`
- `/usr/share/containers/systemd/`

而无 root 文件可以放置在以下任一位置：

- `$XDG_CONFIG_HOME/containers/systemd/` 或 `~/.config/containers/systemd/`
- `/etc/containers/systemd/users/$(UID)`
- `/etc/containers/systemd/users/`

虽然支持单个容器、pod、镜像、网络、卷和 kube 文件，但让我们聚焦于 Nextcloud 示例。创建一个新文件 `~/.config/containers/systemd/nextcloud.container`，内容如下：

```systemd
[Container]
Image=nextcloud
PublishPort=8080:80
```

还有[很多其他选项](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#container-units-container)可用。

要运行生成器并让 systemd 知道有新服务，运行：

```bash
systemctl --user daemon-reload
```

要启动您的服务，运行：

```bash
systemctl --user start nextcloud.service
```

!!! note

    如果您在有 root 服务的目录中创建文件，请省略 `--user` 标志。

要在系统启动或用户登录时自动运行容器，您可以向 `nextcloud.container` 文件添加另一个部分：

```systemd
[Install]
WantedBy=default.target
```

由于生成的 service 文件被视为临时的，它们无法被 systemd 启用。为缓解此问题，生成器在生成期间手动应用安装，这实际上也启用了这些 service 文件。

还支持其他文件类型：pod、volume、network、image 和 kube。[Pods](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#pod-units-pod) 例如可用于对容器进行分组——生成的 systemd 服务及其依赖关系（在容器之前创建 pod）由 systemd 自动管理。

### 使用 `podman generate systemd`

Podman 还提供 `generate systemd` 子命令。它可用于生成 `systemd` 服务文件。

!!! warning

    `generate systemd` 现已弃用，将不再接收新功能。建议使用 Quadlet。

现在让我们用 Nextcloud 来操作。运行：

```bash
podman ps
```

您将获得正在运行的容器列表：

```bash
04f7553f431a  docker.io/library/nextcloud:latest  apache2-foregroun...  5 minutes ago  Up 5 minutes  0.0.0.0:8080->80/tcp  compassionate_meninsky
```

如上所示，我们的容器名称为 `compassionate_meninsky`。

要为 Nextcloud 容器创建 `systemd` 服务并在重启时启用它，运行以下命令：

```bash
podman generate systemd --name compassionate_meninsky > /usr/lib/systemd/system/nextcloud.service
systemctl enable nextcloud
```

将 `compassionate_meninsky` 替换为您容器分配的名称。

当系统重启时，Nextcloud 将在 Podman 中重新启动。
