---
title: openQA - openqa-clone-job 示例
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

本页面将简要介绍使用 `openqa-clone-job` 命令进行基本和高级作业克隆的概述。

## 系统/访问要求

要完成任一示例，请完成 [openQA - 访问](openqa_access.md) 文档中列出的 API `POST` 访问步骤。

## 基本 `openqa-clone-job`

### 查询 openQA 以获取特定测试或作业

首先，你可能需要向 {{ rc.prod }} openQA 系统查询特定作业或测试的最新作业 ID。openQA 客户端（以下称为 `openqa-cli`）将允许你通过 API 快速完成此操作。以下是一个示例...

```bash
$ openqa-cli api --host http://openqa.rockylinux.org jobs/overview groupid=0 distri=rocky version=9.1 test=install_default_upload latest=1 | jq '.'
[
  {
    "id": 22735,
    "name": "rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit"
  }
]
```

这基本上是在说"给我 {{ rc.prod }} 9.1 最新 `install_default_upload` 测试的作业 ID 和名称"。

### "按原样"克隆作业

有了该作业 ID，你现在可以通过以下方式将该作业直接克隆到你的本地 openQA 开发系统...

```bash
$ openqa-clone-job --from https://openqa.rockylinux.org --skip-download 22735
Cloning children of rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit
Created job #23: rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit -> http://localhost/t23
```

### 基本作业概览

现在你应该在本地实例中有相同的作业在运行...

```bash
$ openqa-cli api jobs/overview
[{"id":23,"name":"rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit"}]
```

### 基本作业详情

```bash
$ openqa-cli api jobs/23 | jq '.'
{
  "job": {
    "assets": {
      "iso": [
        "Rocky-9.1-20221214.1-x86_64-dvd.iso"
      ]
    },
    "assigned_worker_id": 2,
    "blocked_by_id": null,
    "children": {
      "Chained": [],
      "Directly chained": [],
      "Parallel": []
    },
    "clone_id": null,
    "group": "Rocky",
    "group_id": 2,
    "has_parents": 0,
    "id": 23,
    "name": "rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit",
    "parents": {
      "Chained": [],
      "Directly chained": [],
      "Parallel": []
    },
    "parents_ok": 1,
    "priority": 50,
    "result": "none",
    "settings": {
      "ARCH": "x86_64",
      "ARCH_BASE_MACHINE": "64bit",
      "BACKEND": "qemu",
      "BUILD": "20230423-Rocky-9.1-x86_64.0",
      "CLONED_FROM": "https://openqa.rockylinux.org/tests/22735",
      "CURRREL": "9",
      "DEPLOY_UPLOAD_TEST": "install_default_upload",
      "DESKTOP": "gnome",
      "DISTRI": "rocky",
      "FLAVOR": "dvd-iso",
      "HDDSIZEGB": "15",
      "ISO": "Rocky-9.1-20221214.1-x86_64-dvd.iso",
      "LOCATION": "https://download.rockylinux.org/pub/rocky/9.1/BaseOS",
      "MACHINE": "64bit",
      "NAME": "00000023-rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit",
      "NICTYPE_USER_OPTIONS": "net=172.16.2.0/24",
      "PACKAGE_SET": "default",
      "PART_TABLE_TYPE": "mbr",
      "POSTINSTALL": "_collect_data",
      "QEMUCPU": "Nehalem",
      "QEMUCPUS": "2",
      "QEMURAM": "2048",
      "QEMU_HOST_IP": "172.16.2.2",
      "QEMU_VIDEO_DEVICE": "virtio-vga",
      "QEMU_VIRTIO_RNG": "1",
      "STORE_HDD_1": "disk_dvd-iso_64bit.qcow2",
      "TEST": "install_default_upload",
      "TEST_SUITE_NAME": "install_default_upload",
      "TEST_TARGET": "ISO",
      "VERSION": "9.1",
      "WORKER_CLASS": "qemu_x86_64",
      "XRES": "1024",
      "YRES": "768"
    },
    "state": "running",
    "t_finished": null,
    "t_started": "2023-04-23T03:02:06",
    "test": "install_default_upload"
  }
}
```

**注意：在上述作业信息中，你可以清楚地看到该作业是从 `https://openqa.rockylinux.org/tests/22735` 克隆的。**

## 高级 `openqa-clone-job`

当然，在从本地实例或生产实例克隆作业时，你可以执行更复杂的操作。通常，这样做可能是为了在克隆作业中修改某些作业 POST 变量，同时保持其他所有变量不变。

### 在克隆时更改变量

以下是一个在克隆作业中更改 ISO 的示例...

```bash
$ openqa-clone-job --from https://openqa.rockylinux.org --skip-download 22735 ISO=Rocky-9.1-x86_64-dvd.iso
Cloning children of rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit
Created job #24: rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit -> http://localhost/t24
```

### 作业概览

```bash
$ openqa-cli api jobs/overview
[{"id":24,"name":"rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit"}]
```

### 作业详情

```bash
$ openqa-cli api jobs/24 | jq '.'
{
  "job": {
    "assets": {
      "iso": [
        "Rocky-9.1-x86_64-dvd.iso"
      ]
    },
    "assigned_worker_id": 1,
    "blocked_by_id": null,
    "children": {
      "Chained": [],
      "Directly chained": [],
      "Parallel": []
    },
    "clone_id": null,
    "group": "Rocky",
    "group_id": 2,
    "has_parents": 0,
    "id": 24,
    "name": "rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit",
    "parents": {
      "Chained": [],
      "Directly chained": [],
      "Parallel": []
    },
    "parents_ok": 1,
    "priority": 50,
    "result": "none",
    "settings": {
      "ARCH": "x86_64",
      "ARCH_BASE_MACHINE": "64bit",
      "BACKEND": "qemu",
      "BUILD": "20230423-Rocky-9.1-x86_64.0",
      "CLONED_FROM": "https://openqa.rockylinux.org/tests/22735",
      "CURRREL": "9",
      "DEPLOY_UPLOAD_TEST": "install_default_upload",
      "DESKTOP": "gnome",
      "DISTRI": "rocky",
      "FLAVOR": "dvd-iso",
      "HDDSIZEGB": "15",
      "ISO": "Rocky-9.1-x86_64-dvd.iso",
      "LOCATION": "https://download.rockylinux.org/pub/rocky/9.1/BaseOS",
      "MACHINE": "64bit",
      "NAME": "00000024-rocky-9.1-dvd-iso-x86_64-Build20230423-Rocky-9.1-x86_64.0-install_default_upload@64bit",
      "NICTYPE_USER_OPTIONS": "net=172.16.2.0/24",
      "PACKAGE_SET": "default",
      "PART_TABLE_TYPE": "mbr",
      "POSTINSTALL": "_collect_data",
      "QEMUCPU": "Nehalem",
      "QEMUCPUS": "2",
      "QEMURAM": "2048",
      "QEMU_HOST_IP": "172.16.2.2",
      "QEMU_VIDEO_DEVICE": "virtio-vga",
      "QEMU_VIRTIO_RNG": "1",
      "STORE_HDD_1": "disk_dvd-iso_64bit.qcow2",
      "TEST": "install_default_upload",
      "TEST_SUITE_NAME": "install_default_upload",
      "TEST_TARGET": "ISO",
      "VERSION": "9.1",
      "WORKER_CLASS": "qemu_x86_64",
      "XRES": "1024",
      "YRES": "768"
    },
    "state": "running",
    "t_finished": null,
    "t_started": "2023-04-23T03:08:03",
    "test": "install_default_upload"
  }
}
```

## 基本和高级 `openqa-clone-job` 的区别

你应该注意到，两个克隆作业之间唯一的实质区别是用于运行 `install_default_upload` 测试的 ISO 不同...

```bash
$ openqa-cli api jobs/23 | jq '.job.settings.ISO'
"Rocky-9.1-20221214.1-x86_64-dvd.iso"

$ openqa-cli api jobs/24 | jq '.job.settings.ISO'
"Rocky-9.1-x86_64-dvd.iso"
```

## 参考资料

[openQA 文档](http://open.qa/documentation/)

{% include 'teams/testing/content_bottom.md' %}
