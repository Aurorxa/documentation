---
title: Podman 入门指南
author: Steven Spencer
contributors: Ganna Zhyrnova
tested_with: 8.5, 8.6, 9.0
tags:
  - containers
  - podman
---

## 引言

Podman 是由 Red Hat 开发的一个容器管理工具。它的 [架构](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods) 驱动了这些功能设计，部分原因在于它使用了 fork-exec 模型。如果您不了解 fork-exec（你并不孤单），简单来说，它允许容器成为用户身份，并且实际上以那个用户的身份运行，这一设计极大地提高了安全性。

Podman 的另一个优势在于，它不需要 root 用户即可运行容器（rootless 模式）。这使其能够与用户命名空间(user namespaces)协同工作，从而为进程提供更细粒度的安全隔离。

Podman 与 Docker 在命令行上是兼容的。两者有许多相似的子命令，但存在一些区别。使用 Podman，您可以在同一台机器上管理 pod 和容器。

## 为什么选择 Podman？

如果一个应用程序在 Docker 中无法正常运行，您可以随时迁移到 Podman。但不管怎样，Podman 是目前容器领域最安全的选择。我们在本系列中关注 Podman 的原因之一就是它是 Rocky Linux 附带的默认容器化工具。

## 安装 Podman

执行以下命令：

```bash
dnf install podman
```

## 容器命令

容器命令类似于 Docker。实际上，您可以在 Podman 中像在 Docker 中一样使用许多命令，并且语法完全相同。

### 拉取镜像(Pulling Images)

在开始创建第一个容器之前，您需要获取一个镜像，然后 Podman 才能进行相关操作。

```bash
podman pull nginx
```

### 创建容器

现在，您可以使用刚拉取的 Nginx 镜像创建一个容器。使用 `-dt` 选项（分离模式）运行容器：

```bash
podman run -dt -p 8080:80/tcp --name nginx_container nginx
```

此命令需要在防火墙中为端口 8080 添加一条规则，该端口转发到容器内的 80 端口。查看端口 `8080` 上的联网内容：

```bash
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

完成后，使用浏览器查看您的 Web 服务器：

![正在运行的 Nginx 容器](../images/nginx_container.png)

点个赞。

列出容器以确保一切正常。

```bash
podman ps -a
```

结果如下所示：

```html
CONTAINER ID  IMAGE                           COMMAND               CREATED        STATUS            PORTS                 NAMES
f1d706f72d3a  docker.io/library/nginx:latest  nginx -g daemon o...  3 seconds ago  Up 4 seconds ago  0.0.0.0:8080->80/tcp  nginx_container
```

要进入容器内部，请使用：

```bash
podman exec -it nginx_container /bin/bash
```

这将给您一个 bash shell。您不再需要直接使用 `ssh` 来连接。

### 管理容器

#### 停止容器

要停止当前正在运行的容器：

```bash
podman stop nginx_container
```

#### 启动容器

当它停止后，您可以通过输入以下命令来启动它：

```bash
podman start nginx_container
```

#### 移除容器

要移除该容器：

```bash
podman stop nginx_container && podman rm nginx_container
```

这会停止容器并删除它。如果您在过去使用过 Docker，那么您可能已经熟悉这个命令。

## 镜像命令

拉取镜像只是第一步，您也可以列出和管理镜像。

#### 列出镜像

要列出本地拉取的镜像，请运行：

```bash
podman images
```

#### 删除镜像

要删除镜像，您可以使用 `rmi` 标志：

```bash
podman rmi docker.io/library/nginx
```

## 卷(Volumes)

卷非常适合用来存储持久化数据，以防容器因某种原因被中止。

要使用卷，您可以执行：

```bash
podman run -dt -v  vol_name:/path/in/container:z image_name
```

### `:z` 和 `:Z` 选项的作用是什么？

我们刚提到的 `:z` 选项究竟是什么意思？它以及 `:Z` 是 SELinux 中用于挂载卷的模式。

* `:z`：使用共享标签(shared label)来表示该卷可在所有容器之间共享。
* `:Z`：使用私有标签(private label)来挂载卷，因此只有您的容器能使用它。

通常 `:z` 选项就足够了。

与 Docker 相比，Podman 的卷提供了更大的灵活性。

## 端口

列出容器上的端口映射：

```bash
podman port -a
```

## Pod 管理

Pod 是共享资源的容器集合。在运行一个容器时，Podman 或 Docker 只要求您运行单个容器，但 Pod 允许您一次性管理多个容器作为一个单元。

例如，如某个应用需要您运行多个容器，您可以创建一个 Pod，这样就不必为每个容器单独运行 `podman run` 命令了。Pod 是 Kubernetes 中常见的管理单位，而 Podman 借用了这种模型。

#### 创建 Pod

```bash
podman pod create --name my_pod
```

要列出 Pod：

```bash
podman pod list
```

上述命令将返回有关 Pod 的基本信息。如果在其中运行一个容器，您会注意到 `STATUS` 从 `Created` 变为 `Running`，但您还必须使用以下命令创建并启动容器：

```bash
podman run -dt --pod my_pod image_name
```

#### 查看 Pod 详细信息

查看 Pod 的详细信息：

```bash
podman pod inspect my_pod
```

#### 移除 Pod

移除 Pod：

```bash
podman pod rm my_pod
```

## podman-compose

如果你偏爱 `docker compose` 的工作方式，那么 `podman-compose` 将会是你的菜。你可以在 Rocky Linux 上通过 EPEL 轻松安装它。

```bash
dnf install podman-compose
```

`podman-compose` 使用 Docker compose 文件，并且运行方式与 `docker compose` 相同。

## 结论

Podman 作为一个比 Docker 更加安全的容器化工具，能够以无守护进程(daemonless)和无根(rootless)的方式运行。使用 Podman 管理容器非常简单，尤其适合那些已经深入了解 Docker 或使用 Docker 的用户。
