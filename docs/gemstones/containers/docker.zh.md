---
title: Docker - 安装引擎
author: Wale Soyinka
contributors: Neel Chauhan, Srinivas Nishant Viswanadha, Stein Arne Storslett, Ganna Zhyrnova, Steven Spencer
date: 2021-08-04
tags:
  - docker
---

# 简介

Docker Engine 可以在 Rocky Linux 服务器上运行原生 Docker 风格的容器工作负载。有时人们更倾向于运行完整的 Docker Desktop 环境之外的这种方式。

## 添加 Docker 仓库

使用 `dnf` 工具将 Docker 仓库添加到您的 Rocky Linux 服务器。输入：

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

## 安装需要的软件包

安装最新版本的 Docker Engine、`containerd` 和 Docker Compose，运行：

```bash
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 启动并启用 Docker (`dockerd`)

使用 `systemctl` 配置 Docker 使其在重启时自动启动，同时立即启动它。输入：

```bash
sudo systemctl --now enable docker
```

## 可选：允许非 root 用户管理 docker

将非 root 用户添加到 `docker` 组，以允许该用户在不使用 `sudo` 的情况下管理 `docker`。

这是一个可选步骤，但如果您是系统的主要用户，或者想允许多个用户管理 docker 但不想授予他们 `sudo` 权限，这会很方便。

输入：

```bash
# 添加当前用户
sudo usermod -a -G docker $(whoami)

# 添加特定用户
sudo usermod -a -G docker custom-user
```

需要注销并重新登录才能被分配新组。使用 `id` 命令检查是否已添加该组。

### 备注

```docker
docker-ce               : 该软件包提供构建和运行 docker 容器 (dockerd) 的底层技术
docker-ce-cli           : 提供命令行界面 (CLI) 客户端 docker 工具 (docker)
containerd.io           : 提供容器运行时 (runc)
docker-buildx-plugin    : Docker CLI 的 Docker Buildx 插件
docker-compose-plugin   : 提供 'docker compose' 子命令的插件
```
