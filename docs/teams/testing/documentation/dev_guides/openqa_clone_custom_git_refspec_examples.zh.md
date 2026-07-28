---
title: openQA - openqa-clone-custom-refspec 示例
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

本页面将简要介绍使用 `openqa-clone-custom-git-refspec` 命令进行基本和高级作业克隆的概述。

从高层次来看，`openqa-clone-custom-git-refspec` 可以被视为一种在不更改默认配置的情况下，直接在 {{ rc.prod }} openQA 实例中测试 openQA 测试的 PR（Pull Request）的机制。因此，它可以支持测试修改测试代码和 needles 的 PR，只要不需要同时修改 `templates.fif.json`。对于 `POST` 变量在 `templates.fif.json` 中预定义的某些情况，可以结合使用 `openqa-clone-custom-git-refspec` 和 `openqa-clone-job`（实际上 `openqa-clone-custom-git-refspec` 在底层使用了 `openqa-clone-job`）。

## 系统/访问要求

要完成任一示例，请完成 [openQA - 访问](openqa_access.md) 文档中列出的 API `POST` 访问步骤。

## 基本 `openqa-clone-custom-git-refspec`

以下示例演示了在 {{ rc.prod }} openQA 生产系统中测试一个开放的 Github Pull Request。该 PR 仅修改测试代码，不提供测试的更新 needles。

### Github PR 信息（基本示例）

***注意：本指南中使用 Github CLI 工具 (`gh`) 来静态展示 PR 信息。***

```text
➜  os-autoinst-distri-rocky git:(develop) gh pr view 168
Serial console install #168
Merged • AlanMarshall wants to merge 1 commit into develop from serial_console • about 27 days ago
+5 -2 • No checks
Reviewers: akatch (Approved), tcooper (Approved), lumarel (Requested)
Labels: priority: medium, type: bug, test suite


  Network is enabled by default at v9 so requires conditional code to handle multiple versions.
  Tested for 9.1, 8.7 & 8.8:

    openqa-cli api -X POST isos ISO=Rocky-9.1-20221214.1-x86_64-dvd.iso ARCH=x86_64 DISTRI=rocky FLAVOR=universal
  VERSION=9.1 BUILD=-"$(date +%Y%m%d.%H%M%S).0"-9.1-20221214.1-universal TEST=install_serial_console
    openqa-cli api -X POST isos ISO=Rocky-8.7-x86_64-dvd1.iso ARCH=x86_64 DISTRI=rocky FLAVOR=universal VERSION=8.7 BUILD=-
  "$(date +%Y%m%d.%H%M%S).0"-8.7-20221110-universal TEST=install_serial_console
    openqa-cli api -X POST isos ISO=Rocky-8.8-x86_64-dvd1.iso ARCH=x86_64 DISTRI=rocky FLAVOR=universal VERSION=8.8 BUILD=-
  "$(date +%Y%m%d.%H%M%S).0"-8.8-lookahead-universal TEST=install_serial_console

  Result: Tests pass.
  Also confirm that all main hub check boxes are checked and user test created prior to start of installation.
  Fixes Issue #102

View this pull request on GitHub: https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/168
```

以上是原始 PR 中提供的信息，包含 Alan 在其 openQA 开发系统中执行的测试。我们可以在确定每个要测试的 Rocky Linux 版本的适当作业 ID 后，在 {{ rc.prod }} openQA 系统中重新运行失败的测试。在此示例中，使用 openQA WebUI 来查找要克隆的适当测试 ID。

### 以 `--verbose --dry-run` 模式运行 `openqa-clone-custom-git-refspec`（基本示例）

实践中，即便对于基本用例，以 `--verbose` 和 `--dry-run` 模式运行 `openqa-clone-custom-git-refspec` 以观察其行为也是有用的...

```bash
$ openqa-clone-custom-git-refspec --verbose --dry-run \
    https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/168 \
    https://openqa.rockylinux.org/tests/16080 2>&1 | tee pr-168
```

***注意：此处不会展示 `openqa-clone-custom-git-refspec` 的完整输出。***

```diff
+ shift
+ true
+ case "$1" in
+ dry_run=true
+ shift
+ true
+ case "$1" in
+ shift
+ break
+ job_list=https://openqa.rockylinux.org/tests/16080
+ [[ -z '' ]]
+ first_arg=https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/168
+ [[ https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/168 == *\p\u\l\l* ]]
+ pr_url=https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/168
+ target_repo_part=https://github.com/rocky-linux/os-autoinst-distri-rocky
+ pr=168
+ pr=168
+ [[ -z '' ]]
+ pr_url=https://api.github.com/repos/rocky-linux/os-autoinst-distri-rocky/pulls/168
++ eval 'curl -s https://api.github.com/repos/rocky-linux/os-autoinst-distri-rocky/pulls/168'
+++ curl -s https://api.github.com/repos/rocky-linux/os-autoinst-distri-rocky/pulls/168

...<snip>...

++ jq -r '.NEEDLES_DIR | select (.!=null)'
+ old_needledir=
+ local needles_dir=
+ needles_dir=rocky/needles
+ local repo_branch=AlanMarshall/os-autoinst-distri-rocky#serial_console
+ local test_suffix=@AlanMarshall/os-autoinst-distri-rocky#serial_console
+ local build=AlanMarshall/os-autoinst-distri-rocky#168
+ local casedir=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#serial_console
+ local GROUP=0
+ local dry_run=true
+ local scriptdir
++ dirname /usr/bin/openqa-clone-custom-git-refspec
+ scriptdir=/usr/bin
+ local 'cmd=true /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance "https://openqa.rockylinux.org" "15973" _GROUP="0" TEST+="@AlanMarshall/os-autoinst-distri-rocky#serial_console" BUILD="AlanMarshall/os-autoinst-distri-rocky#168" CASEDIR="https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#serial_console" PRODUCTDIR="os-autoinst-distri-rocky" NEEDLES_DIR="rocky/needles"'
+ [[ 0 -ne 0 ]]
+ [[ -n '' ]]
+ eval 'true /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance "https://openqa.rockylinux.org" "15973" _GROUP="0" TEST+="@AlanMarshall/os-autoinst-distri-rocky#serial_console" BUILD="AlanMarshall/os-autoinst-distri-rocky#168" CASEDIR="https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#serial_console" PRODUCTDIR="os-autoinst-distri-rocky" NEEDLES_DIR="rocky/needles"'
++ true /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance https://openqa.rockylinux.org 15973 _GROUP=0 TEST+=@AlanMarshall/os-autoinst-distri-rocky#serial_console BUILD=AlanMarshall/os-autoinst-distri-rocky#168 CASEDIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#serial_console PRODUCTDIR=os-autoinst-distri-rocky NEEDLES_DIR=rocky/needles
```

从 `openqa-clone-custom-git-refspec` 的完整 `--dry-run` 输出中可以看出，要克隆的作业和要使用的 PR 都被检查，并生成一个 `openqa-clone-job` 命令以提交到作业正在被克隆到的 openQA 系统。

如果不使用 `--dry-run`，上面显示的最终 `openqa-clone-job` 命令将被运行，导致目标作业被克隆，并带有额外的 `POST` 变量，这些变量将使 PR 中引用的仓库/分支被克隆到测试目录中，并在克隆的作业中引用重要的文件。

### 不带 `--verbose --dry-run` 模式运行 `openqa-clone-custom-git-refspec`

```bash
$ openqa-clone-custom-git-refspec \
    https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/168 \
    https://openqa.rockylinux.org/tests/16080
Created job #16119: rocky-9.1-universal-x86_64-Build20230329-Rocky-9.1-x86_64.0-install_serial_console@64bit -> https://openqa.rockylinux.org/t16119
```

### 克隆作业信息

```bash
$ openqa-cli api jobs/16119 --pretty
{
   "job" : {
      "assets" : {
         "iso" : [
            "Rocky-9.1-20221214.1-x86_64-dvd.iso"
         ]
      },
      "assigned_worker_id" : 5,
      "blocked_by_id" : null,
      "children" : {
         "Chained" : [],
         "Directly chained" : [],
         "Parallel" : []
      },
      "clone_id" : 16121,
      "group_id" : null,
      "has_parents" : 0,
      "id" : 16119,
      "name" : "rocky-9.1-universal-x86_64-BuildAlanMarshall_os-autoinst-distri-rocky_168-install_serial_console@AlanMarshall_os-autoinst-distri-rocky_serial_console@64bit",
      "parents" : {
         "Chained" : [],
         "Directly chained" : [],
         "Parallel" : []
      },
      "parents_ok" : 1,
      "priority" : 10,
      "reason" : "isotovideo abort: isotovideo received signal TERM",
      "result" : "user_restarted",
      "settings" : {
         "ANACONDA_TEXT" : "1",
         "ARCH" : "x86_64",
         "ARCH_BASE_MACHINE" : "64bit",
         "BACKEND" : "qemu",
         "BUILD" : "AlanMarshall\/os-autoinst-distri-rocky#168",
         "CASEDIR" : "https:\/\/github.com\/AlanMarshall\/os-autoinst-distri-rocky.git#serial_console",
         "CLONED_FROM" : "https:\/\/openqa.rockylinux.org\/tests\/15973",
         "CURRREL" : "9",
         "DISTRI" : "rocky",
         "FLAVOR" : "universal",
         "HDDSIZEGB" : "15",
         "ISO" : "Rocky-9.1-20221214.1-x86_64-dvd.iso",
         "LOCATION" : "https:\/\/download.rockylinux.org\/pub\/rocky\/9.1\/BaseOS",
         "MACHINE" : "64bit",
         "NAME" : "00016119-rocky-9.1-universal-x86_64-BuildAlanMarshall_os-autoinst-distri-rocky_168-install_serial_console@AlanMarshall_os-autoinst-distri-rocky_serial_console@64bit",
         "NEEDLES_DIR" : "rocky\/needles",
         "NICTYPE_USER_OPTIONS" : "net=172.16.2.0\/24",
         "NO_UEFI_POST" : "1",
         "PART_TABLE_TYPE" : "mbr",
         "PRODUCTDIR" : "os-autoinst-distri-rocky",
         "QEMUCPU" : "Nehalem",
         "QEMUCPUS" : "2",
         "QEMURAM" : "2048",
         "QEMU_HOST_IP" : "172.16.2.2",
         "QEMU_VIDEO_DEVICE" : "virtio-vga",
         "QEMU_VIRTIO_RNG" : "1",
         "SERIAL_CONSOLE" : "1",
         "TEST" : "install_serial_console@AlanMarshall\/os-autoinst-distri-rocky#serial_console",
         "TEST_SUITE_NAME" : "install_serial_console",
         "TEST_TARGET" : "ISO",
         "VERSION" : "9.1",
         "VIRTIO_CONSOLE_NUM" : "2",
         "WORKER_CLASS" : "qemu_x86_64",
         "XRES" : "1024",
         "YRES" : "768"
      },
      "state" : "done",
      "t_finished" : "2023-03-29T06:19:37",
      "t_started" : "2023-03-29T06:12:26",
      "test" : "install_serial_console@AlanMarshall\/os-autoinst-distri-rocky#serial_console"
   }
}
```

## 高级 `openqa-clone-custom-git-refspec`

以下示例演示了在 {{ rc.prod }} openQA 生产系统中测试一个开放的 Github Pull Request。该 PR 修改测试代码并提供测试的更新 needles。

### Github PR 信息（高级示例）

```text
➜  os-autoinst-distri-rocky git:(nazunalika/develop) gh pr view 162

Anaconda text install #162
Open • AlanMarshall wants to merge 2 commits into develop from anaconda-txt • about 1 day ago
+30 -5 • No checks
Reviewers: akatch (Approved), lumarel (Requested), tcooper (Requested)
Labels: priority: medium, type: bug, test suite


  Added new needle for text install.
  Deleted redundant code.
  Tested for 9.1, 8.7 & 8.8:

    openqa-cli api -X POST isos ISO=Rocky-9.1-20221214.1-x86_64-dvd.iso ARCH=x86_64 DISTRI=rocky FLAVOR=universal
  VERSION=9.1 BUILD=-"$(date +%Y%m%d.%H%M%S).0"-9.1-20221214.1-universal TEST=install_anaconda_text
    openqa-cli api -X POST isos ISO=Rocky-8.7-x86_64-dvd1.iso ARCH=x86_64 DISTRI=rocky FLAVOR=universal VERSION=8.7 BUILD=-
  "$(date +%Y%m%d.%H%M%S).0"-8.7-20221110-universal TEST=install_anaconda_text
    openqa-cli api -X POST isos ISO=Rocky-8.8-x86_64-dvd1.iso ARCH=x86_64 DISTRI=rocky FLAVOR=universal VERSION=8.8 BUILD=-
  "$(date +%Y%m%d.%H%M%S).0"-8.8-lookahead-universal TEST=install_anaconda_text

  Result: Pass
  Fixes Issue #145


akatch approved (Member) • 18h • Newest comment

  All indicated tests pass.


View this pull request on GitHub: https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/162
```

### 以 `--verbose --dry-run` 模式运行 `openqa-clone-custom-git-refspec`（高级示例）

```diff
$ openqa-clone-custom-git-refspec --verbose --dry-run https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/162 https://openqa.rockylinux.org/tests/13371
+ shift
+ true
+ case "$1" in
+ dry_run=true
+ shift
+ true
+ case "$1" in
+ shift
+ break
+ job_list=https://openqa.rockylinux.org/tests/13371
+ [[ -z '' ]]
+ first_arg=https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/162
+ [[ https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/162 == *\p\u\l\l* ]]
+ pr_url=https://github.com/rocky-linux/os-autoinst-distri-rocky/pull/162
+ target_repo_part=https://github.com/rocky-linux/os-autoinst-distri-rocky


...<snip>...

++ jq -r '.NEEDLES_DIR | select (.!=null)'
+ old_needledir=
+ local needles_dir=
+ needles_dir=rocky/needles
+ local repo_branch=AlanMarshall/os-autoinst-distri-rocky#anaconda-txt
+ local test_suffix=@AlanMarshall/os-autoinst-distri-rocky#anaconda-txt
+ local build=AlanMarshall/os-autoinst-distri-rocky#162
+ local casedir=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt
+ local GROUP=0
+ local dry_run=true
+ local scriptdir
++ dirname /usr/bin/openqa-clone-custom-git-refspec
+ scriptdir=/usr/bin
+ local 'cmd=true /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance "https://openqa.rockylinux.org" "13371" _GROUP="0" TEST+="@AlanMarshall/os-autoinst-distri-rocky#anaconda-txt" BUILD="AlanMarshall/os-autoinst-distri-rocky#162" CASEDIR="https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt" PRODUCTDIR="os-autoinst-distri-rocky" NEEDLES_DIR="rocky/needles"'
+ [[ 0 -ne 0 ]]
+ [[ -n '' ]]
+ eval 'true /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance "https://openqa.rockylinux.org" "13371" _GROUP="0" TEST+="@AlanMarshall/os-autoinst-distri-rocky#anaconda-txt" BUILD="AlanMarshall/os-autoinst-distri-rocky#162" CASEDIR="https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt" PRODUCTDIR="os-autoinst-distri-rocky" NEEDLES_DIR="rocky/needles"'
++ true /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance https://openqa.rockylinux.org 13371 _GROUP=0 TEST+=@AlanMarshall/os-autoinst-distri-rocky#anaconda-txt BUILD=AlanMarshall/os-autoinst-distri-rocky#162 CASEDIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt PRODUCTDIR=os-autoinst-distri-rocky NEEDLES_DIR=rocky/needles
```

此 PR 提供了更新的 needles，而 `openqa-clone-custom-git-refspec` 的默认行为是**不**为 `NEEDLES` 提供备用位置。需要修改 `--verbose --dry-run` 输出以确保 PR 中提供的 needles 在测试中被使用。

### 修改 `--verbose --dry-run` 输出以指向 PR 中的 needles

使用输出修改克隆作业...

#### 原始

```bash
$ /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance https://openqa.rockylinux.org \
  13371 _GROUP=0 TEST+=@AlanMarshall/os-autoinst-distri-rocky#anaconda-txt \
  BUILD=AlanMarshall/os-autoinst-distri-rocky#162 CASEDIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt \
  PRODUCTDIR=os-autoinst-distri-rocky
NEEDLES_DIR=rocky/needles
```

#### 手动指定 NEEDLES_DIR 指向 PR 分支

```bash
$ /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance https://openqa.rockylinux.org \
  13371 _GROUP=0 TEST+=@AlanMarshall/os-autoinst-distri-rocky#anaconda-txt \
  BUILD=AlanMarshall/os-autoinst-distri-rocky#162 CASEDIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt \
  PRODUCTDIR=os-autoinst-distri-rocky NEEDLES_DIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt/needles
```

#### {{ rc.prod }} 9.1

```bash
$ /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance https://openqa.rockylinux.org \
  13255 _GROUP=0 TEST+=@AlanMarshall/os-autoinst-distri-rocky#anaconda-txt \
  BUILD=AlanMarshall/os-autoinst-distri-rocky#162 CASEDIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt \
  PRODUCTDIR=os-autoinst-distri-rocky NEEDLES_DIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt/needles
Created job #14228: rocky-9.1-universal-x86_64-Build20230319-Rocky-9.1-x86_64.0-install_anaconda_text@64bit -> https://openqa.rockylinux.org/t14228
```

```bash
$ openqa-cli api jobs/14228 --pretty
{
   "job" : {
      "assets" : {
         "iso" : [
            "Rocky-9.1-20221214.1-x86_64-dvd.iso"
         ]
      },
      "assigned_worker_id" : 9,
      "blocked_by_id" : null,
      "children" : {
         "Chained" : [],
         "Directly chained" : [],
         "Parallel" : []
      },
      "clone_id" : null,
      "group_id" : null,
      "has_parents" : 0,
      "id" : 14228,
      "name" : "rocky-9.1-universal-x86_64-BuildAlanMarshall_os-autoinst-distri-rocky_162-install_anaconda_text@AlanMarshall_os-autoinst-distri-rocky_anaconda-txt@64bit",
      "parents" : {
         "Chained" : [],
         "Directly chained" : [],
         "Parallel" : []
      },
      "parents_ok" : 1,
      "priority" : 0,
      "result" : "passed",
      "settings" : {
         "ANACONDA_TEXT" : "1",
         "ARCH" : "x86_64",
         "ARCH_BASE_MACHINE" : "64bit",
         "BACKEND" : "qemu",
         "BUILD" : "AlanMarshall\/os-autoinst-distri-rocky#162",
         "CASEDIR" : "https:\/\/github.com\/AlanMarshall\/os-autoinst-distri-rocky.git#anaconda-txt",
         "CLONED_FROM" : "https:\/\/openqa.rockylinux.org\/tests\/13255",
         "CURRREL" : "9",
         "DISTRI" : "rocky",
         "FLAVOR" : "universal",
         "HDDSIZEGB" : "15",
         "ISO" : "Rocky-9.1-20221214.1-x86_64-dvd.iso",
         "LOCATION" : "https:\/\/dl.rockylinux.org\/pub\/rocky\/9.1",
         "MACHINE" : "64bit",
         "NAME" : "00014228-rocky-9.1-universal-x86_64-BuildAlanMarshall_os-autoinst-distri-rocky_162-install_anaconda_text@AlanMarshall_os-autoinst-distri-rocky_anaconda-txt@64bit",
         "NEEDLES_DIR" : "https:\/\/github.com\/AlanMarshall\/os-autoinst-distri-rocky.git#anaconda-txt\/needles",
         "NICTYPE_USER_OPTIONS" : "net=172.16.2.0\/24",
         "PART_TABLE_TYPE" : "mbr",
         "PRODUCTDIR" : "os-autoinst-distri-rocky",
         "QEMUCPU" : "Nehalem",
         "QEMUCPUS" : "2",
         "QEMURAM" : "2048",
         "QEMU_HOST_IP" : "172.16.2.2",
         "QEMU_VIDEO_DEVICE" : "virtio-vga",
         "QEMU_VIRTIO_RNG" : "1",
         "TEST" : "install_anaconda_text@AlanMarshall\/os-autoinst-distri-rocky#anaconda-txt",
         "TEST_SUITE_NAME" : "install_anaconda_text",
         "TEST_TARGET" : "ISO",
         "VERSION" : "9.1",
         "WORKER_CLASS" : "qemu_x86_64",
         "XRES" : "1024",
         "YRES" : "768"
      },
      "state" : "done",
      "t_finished" : "2023-03-22T05:28:28",
      "t_started" : "2023-03-22T05:07:09",
      "test" : "install_anaconda_text@AlanMarshall\/os-autoinst-distri-rocky#anaconda-txt"
   }
}
```

![openqa-clone-custome-git-refspec-job-14228 示例...](../../../../assets/teams/testing/openqa-clone-custom-git-refspec-job-14228.png){ loading=lazy }

#### {{ rc.prod }} 8.7

```bash
$ /usr/bin/openqa-clone-job --skip-chained-deps --parental-inheritance --within-instance https://openqa.rockylinux.org \
  13371 _GROUP=0 TEST+=@AlanMarshall/os-autoinst-distri-rocky#anaconda-txt \
  BUILD=AlanMarshall/os-autoinst-distri-rocky#162 CASEDIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt \
  PRODUCTDIR=os-autoinst-distri-rocky NEEDLES_DIR=https://github.com/AlanMarshall/os-autoinst-distri-rocky.git#anaconda-txt/needles
Created job #14229: rocky-8.7-universal-x86_64-Build20230319-Rocky-8.7-x86_64.0-install_anaconda_text@64bit -> https://openqa.rockylinux.org/t14229
```

```bash
$ openqa-cli api jobs/14229 --pretty
{
   "job" : {
      "assets" : {
         "iso" : [
            "Rocky-8.7-x86_64-dvd1.iso"
         ]
      },
      "assigned_worker_id" : 8,
      "blocked_by_id" : null,
      "children" : {
         "Chained" : [],
         "Directly chained" : [],
         "Parallel" : []
      },
      "clone_id" : null,
      "group_id" : null,
      "has_parents" : 0,
      "id" : 14229,
      "name" : "rocky-8.7-universal-x86_64-BuildAlanMarshall_os-autoinst-distri-rocky_162-install_anaconda_text@AlanMarshall_os-autoinst-distri-rocky_anaconda-txt@64bit",
      "parents" : {
         "Chained" : [],
         "Directly chained" : [],
         "Parallel" : []
      },
      "parents_ok" : 1,
      "priority" : 0,
      "result" : "passed",
      "settings" : {
         "ANACONDA_TEXT" : "1",
         "ARCH" : "x86_64",
         "ARCH_BASE_MACHINE" : "64bit",
         "BACKEND" : "qemu",
         "BUILD" : "AlanMarshall\/os-autoinst-distri-rocky#162",
         "CASEDIR" : "https:\/\/github.com\/AlanMarshall\/os-autoinst-distri-rocky.git#anaconda-txt",
         "CLONED_FROM" : "https:\/\/openqa.rockylinux.org\/tests\/13371",
         "CURRREL" : "8",
         "DISTRI" : "rocky",
         "FLAVOR" : "universal",
         "HDDSIZEGB" : "15",
         "ISO" : "Rocky-8.7-x86_64-dvd1.iso",
         "LOCATION" : "https:\/\/dl.rockylinux.org\/pub\/rocky\/8.7",
         "MACHINE" : "64bit",
         "NAME" : "00014229-rocky-8.7-universal-x86_64-BuildAlanMarshall_os-autoinst-distri-rocky_162-install_anaconda_text@AlanMarshall_os-autoinst-distri-rocky_anaconda-txt@64bit",
         "NEEDLES_DIR" : "https:\/\/github.com\/AlanMarshall\/os-autoinst-distri-rocky.git#anaconda-txt\/needles",
         "NICTYPE_USER_OPTIONS" : "net=172.16.2.0\/24",
         "PART_TABLE_TYPE" : "mbr",
         "PRODUCTDIR" : "os-autoinst-distri-rocky",
         "QEMUCPU" : "Nehalem",
         "QEMUCPUS" : "2",
         "QEMURAM" : "2048",
         "QEMU_HOST_IP" : "172.16.2.2",
         "QEMU_VIDEO_DEVICE" : "virtio-vga",
         "QEMU_VIRTIO_RNG" : "1",
         "TEST" : "install_anaconda_text@AlanMarshall\/os-autoinst-distri-rocky#anaconda-txt",
         "TEST_SUITE_NAME" : "install_anaconda_text",
         "TEST_TARGET" : "ISO",
         "VERSION" : "8.7",
         "WORKER_CLASS" : "qemu_x86_64",
         "XRES" : "1024",
         "YRES" : "768"
      },
      "state" : "done",
      "t_finished" : "2023-03-22T05:31:22",
      "t_started" : "2023-03-22T05:10:46",
      "test" : "install_anaconda_text@AlanMarshall\/os-autoinst-distri-rocky#anaconda-txt"
   }
}
```

![openqa-clone-custome-git-refspec-job-14229 示例...](../../../../assets/teams/testing/openqa-clone-custom-git-refspec-job-14229.png){ loading=lazy }

## 参考资料

[openQA 文档](http://open.qa/documentation/)

{% include 'teams/testing/content_bottom.md' %}
