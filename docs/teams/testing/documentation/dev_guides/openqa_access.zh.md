---
title: openQA - Rocky 生产环境访问
author: Trevor Cooper
contributors: Lukas Magauer
tested_with:
tags:
  - testing
  - openQA
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
  level: Final
render_macros: true
---

## 系统要求

要访问 Rocky 生产环境 openQA 系统并完成以下任一示例，你需要访问一个提供 openQA 客户端的系统。通常那将是一个基于 Fedora 的系统/容器（但也可以是 Rocky 9.6），并安装了 `openqa-client` 软件包及其（约 239 个）依赖项。

或者，你可以在自己的本地机器上安装 openQA 服务器。参见：[手动安装](openqa_manual_install.md)

## 访问要求

### API `GET` 访问

{{ rc.prod }} openQA 系统允许通过其 Web 界面或使用 `openqa-client` 对 API 进行 `GET` 操作的不受限制的公开访问。

### API `POST` 访问

要使用 openQA 客户端与 {{ rc.prod }} openQA 系统进行 `POST` 操作的交互，需要满足以下条件：

- 在 [{{ rc.prod }} 账户服务](https://accounts.rockylinux.org) 系统中拥有一个状态良好的账户，
- 从 {{ rc.prod }} 测试团队获得 API `POST` 访问授权，以及
- 在 {{ rc.prod }} openQA 系统上生成的 [openQA API 密钥](https://open.qa/docs/#_authentication)。

## 配置你的 openqa 客户端

根据 openqa 客户端命令帮助，你可以通过多种方式配置客户端以使用你的 API 密钥。

以下示例展示了通过最常用的方法配置你的客户端。通过这种方式可以配置多个 openqa 客户端 API 密钥。

```bash
$ mkdir -p ~/.config/openqa

$ vim ~/.config/openqa/client.conf

$ cat ~/.config/openqa/client.conf
[localhost]
key = your_localhost_api_key
secret = your_localhost_api_secret
[openqa.rockylinux.org]
key = your_api_key
secret = your_api_secret
```

## 测试你的 openqa 客户端安装

```bash
openqa-cli api --host https://openqa.rockylinux.org --pretty jobs/overview
```

应提供当前作业列表，然后选择一个作业编号并查看该特定作业的信息，例如：

```bash
$ openqa-cli api --host https://openqa.rockylinux.org --pretty jobs/1
{
   "job" : {
      "assets" : {
         "iso" : [
            "Rocky-8.6-x86_64-boot.iso"
         ]
      },
      "assigned_worker_id" : 2,
      "blocked_by_id" : null,
      "children" : {
         "Chained" : [],
         "Directly chained" : [],
         "Parallel" : []
      },
      "clone_id" : null,
      "group" : "Rocky",
      "group_id" : 2,
      "has_parents" : 0,
      "id" : 1,
      "name" : "rocky-8.6-boot-iso-x86_64-Build-8.6-boot-iso--20221110.223812.0-install_default@64bit",
      "parents" : {
         "Chained" : [],
         "Directly chained" : [],
         "Parallel" : []
      },
      "parents_ok" : 1,
      "priority" : 10,
      "result" : "failed",
      "settings" : {
         "ARCH" : "x86_64",
         "ARCH_BASE_MACHINE" : "64bit",
         "BACKEND" : "qemu",
         "BUILD" : "-8.6-boot-iso--20221110.223812.0",
         "DESKTOP" : "gnome",
         "DISTRI" : "rocky",
         "FLAVOR" : "boot-iso",
         "GRUB" : "ip=dhcp",
         "HDDSIZEGB" : "15",
         "ISO" : "Rocky-8.6-x86_64-boot.iso",
         "MACHINE" : "64bit",
         "NAME" : "00000001-rocky-8.6-boot-iso-x86_64-Build-8.6-boot-iso--20221110.223812.0-install_default@64bit",
         "PACKAGE_SET" : "default",
         "PART_TABLE_TYPE" : "mbr",
         "POSTINSTALL" : "_collect_data",
         "QEMUCPU" : "Nehalem",
         "QEMUCPUS" : "2",
         "QEMURAM" : "3072",
         "QEMUVGA" : "virtio",
         "QEMU_VIRTIO_RNG" : "1",
         "TEST" : "install_default",
         "TEST_SUITE_NAME" : "install_default",
         "TEST_TARGET" : "ISO",
         "VERSION" : "8.6",
         "WORKER_CLASS" : "qemu_x86_64"
      },
      "state" : "done",
      "t_finished" : "2022-11-10T22:44:19",
      "t_started" : "2022-11-10T22:38:12",
      "test" : "install_default"
   }
}
```

## 参考资料

- [openQA 文档](https://open.qa/documentation/)
- [安装信息](https://github.com/rocky-linux/OpenQA-Fedora-Installation)
- [Rocky 测试](https://git.resf.org/testing/os-autoinst-distri-rocky)

{% include 'teams/testing/content_bottom.md' %}
