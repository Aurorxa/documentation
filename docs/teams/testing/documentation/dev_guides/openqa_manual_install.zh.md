---
title: 为 Rocky Linux 手动安装 openQA
author: Bob Robison
contributors: Trevor Cooper
tested_with:
tags:
  - testing
  - openQA
revision_date: 2026-05-08
version: v1.3
rc:
  prod: Rocky Linux
  level: Issue
render_macros: true
---

## 目标受众

希望使用已配置好 Rocky Linux 测试的 openQA 自动化测试系统的人员。如果如此，你需要一台支持硬件虚拟化的 PC 或服务器，运行最新的 Fedora Linux 或 RockyLinux >= 9.6。

### 介绍

本指南解释如何使用 openQA 自动化测试系统来测试 Rocky Linux 版本的各个方面，无论是在预发布阶段还是之后。

openQA 是一个自动化测试工具，可以测试整个安装过程。它使用虚拟机来重现安装过程，在每个步骤检查输出（串行控制台和 GUI 屏幕），并发送必要的按键和命令以继续下一步。openQA 检查系统是否可以安装，是否正常工作，应用程序能否运行，以及系统对不同安装选项和命令的响应是否符合预期。

Rocky Linux 的 openQA 测试可以在 [os-autoinst-distri-rocky](https://github.com/rocky-linux/os-autoinst-distri-rocky) 仓库中找到。

openQA 可以为操作系统的每个修订版本运行大量测试组合，报告每种硬件配置、安装选项和操作系统变体的组合中检测到的错误。

### WebUI

Web UI 是 openQA 系统的一个非常实用的功能，因为它提供了一个易于访问的视图，可以查看本地或远程（或两者）openQA 测试的进度和详情。它设计得直观且不言自明。

某些页面使用查询来选择应显示的内容。查询参数通过可点击的链接生成，例如从索引页或组概览页点击单个构建。在查询页面上，可以有 UI 元素来控制参数，例如查找较旧的构建或仅显示失败的作业，或其他设置。此外，如果你想提供指向特定视图的链接，可以手动调整查询参数。

## 分步安装指南

openQA 只能安装在 Fedora、OpenSUSE 或 RockyLinux(>=9.6) 服务器或工作站上。以下安装过程在 Fedora 40 Server 上进行了测试。你可以使用本地终端或通过局域网从另一台主机进行 SSH 登录。

```bash
# 安装软件包
# 用于 openqa
sudo dnf install -y openqa openqa-httpd openqa-worker fedora-messaging python3-jsonschema
sudo dnf install -y perl-REST-Client.noarch

# 用于 createhdds
sudo dnf install -y libguestfs-tools libguestfs-xfs python3-fedfind python3-libguestfs
sudo dnf install -y libvirt libvirt-daemon-config-network libvirt-python3 virt-install withlock

# 配置 httpd:
cd /etc/httpd/conf.d/
sudo cp openqa.conf.template openqa.conf
sudo cp openqa-ssl.conf.template openqa-ssl.conf
sudo setsebool -P httpd_can_network_connect 1
sudo systemctl restart httpd

# 配置 Web UI
sudoedit /etc/openqa/openqa.ini
[global]
branding=plain
download_domains = rockylinux.org
[auth]
method = Fake

sudo dnf install postgresql-server
sudo postgresql-setup --initdb

# 启用并启动服务
sudo systemctl enable postgresql --now
sudo systemctl enable httpd --now
sudo systemctl enable openqa-gru --now
sudo systemctl enable openqa-scheduler --now
sudo systemctl enable openqa-websockets --now
sudo systemctl enable openqa-webui --now
sudo systemctl enable fm-consumer@fedora_openqa_scheduler --now
sudo systemctl enable libvirtd --now
sudo setsebool -P httpd_can_network_connect 1
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
sudo systemctl restart httpd

# 在本地 Web 界面 http://localhost 创建 API 密钥
#   或在远程系统 http://<ip addr>
# 点击 Login，然后 Manage API Keys，创建一个密钥和 secret。

# 插入密钥和 secret
sudoedit /etc/openqa/client.conf

[localhost]
key = ...
secret = ...

# 创建 worker
sudo systemctl enable openqa-worker@1 --now
# 然后 ...@2 ... 等根据需要。在 webui 的 workers 中查看以检查是否显示为 idle。
# 一般而言，worker 数量可以约为核心数的一半

# 获取 Rocky 测试
cd /var/lib/openqa/tests/
sudo git clone https://github.com/rocky-linux/os-autoinst-distri-rocky.git rocky
sudo chown -R geekotest:geekotest rocky
cd rocky

# 在 /var/lib/openqa 中工作时，几乎所有命令都需要 sudo。

sudo git config --global --add safe.directory /var/lib/openqa/share/tests/rocky

sudo git checkout develop
# 或者选择包含最新测试更新的分支

sudo ./fifloader.py -l -c templates.fif.json
sudo git clone https://github.com/rocky-linux/createhdds.git  ~/createhdds
sudo mkdir -p /var/lib/openqa/share/factory/hdd/fixed

# 需要约 200GB 磁盘空间用于持续测试
cd /var/lib/openqa/factory/hdd/fixed

# 启动一个长时间运行的进程，为持续测试提供 hdd 镜像文件
~/createhdds/createhdds.py -l info -t all

# 从 staging 仓库获取用于测试的 Rocky iso 文件
sudo mkdir -p /var/lib/openqa/share/factory/iso/fixed && \
    cd /var/lib/openqa/share/factory/iso/fixed

sudo curl -LR -o Rocky-9.7-x86_64-boot.iso \
    --output-dir /var/lib/openqa/share/factory/iso/fixed \
    https://dl.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9-latest-x86_64-boot.iso

sudo curl -LR -o Rocky-9.7-x86_64-minimal.iso \
    --output-dir /var/lib/openqa/share/factory/iso/fixed \
    https://dl.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9-latest-x86_64-minimal.iso

sudo curl -LR -o Rocky-9.7-x86_64-dvd.iso \
    --output-dir /var/lib/openqa/share/factory/iso/fixed \
    https://dl.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9-latest-x86_64-dvd.iso

sudo curl -LR -o CHECKSUM \
    --output-dir /var/lib/openqa/share/factory/iso/fixed \
    https://dl.rockylinux.org/pub/rocky/9/isos/x86_64/CHECKSUM

sha256sum -c CHECKSUM

# 修复所有权，将 <user> 添加到组，重启
cd /var/lib/openqa/factory/
sudo chown -R geekotest:geekotest ./
sudo usermod -aG geekotest <user>
sudo init 6

# 发布测试并在 webui 上查看进度
cd /var/lib/openqa/tests/rocky/
sudo ./fifloader.py -c -l templates.fif.json
sudo openqa-cli api -X POST isos ISO=Rocky-9.7-x86_64-minimal.iso ARCH=x86_64 DISTRI=rocky FLAVOR=minimal-iso VERSION=9.7 BUILD="$(date +%Y%m%d.%H%M%S).0"-minimal
sudo openqa-cli api -X POST isos ISO=Rocky-9.7-x86_64-boot.iso ARCH=x86_64 DISTRI=rocky FLAVOR=boot-iso VERSION=9.7 BUILD="$(date +%Y%m%d.%H%M%S).0"-boot
对于完整构建（将提交 95 个作业）：
sudo openqa-cli api -X POST isos ISO=Rocky-9.7-x86_64-dvd.iso ARCH=x86_64 DISTRI=rocky FLAVOR=dvd-iso VERSION=9.7 BUILD="$(date +%Y%m%d.%H%M%S).0"-dvd-iso
sudo openqa-cli api -X POST isos ISO=Rocky-9.7-x86_64-dvd.iso ARCH=x86_64 DISTRI=rocky FLAVOR=universal VERSION=9.7 BUILD="$(date +%Y%m%d.%H%M%S).0"-universal
```

你可以在与测试主机处于同一局域网的任何浏览器上通过 webui 查看这些测试的进度，地址为：

```http://<测试主机IP地址>/tests```

如果你点击右上角的"login"，你将能够通过 webui 控制测试。

此时，多虚拟机（multi-vm）测试将失败或被跳过。这是因为你的系统目前配置为单虚拟机部署，可以按此方式使用。如果你希望在继续之前先实践使用 openQA，请暂停安装（推荐）。

多虚拟机测试设施的安装要复杂得多，将在本文档的后续修订版本中描述（敬请期待）。

### 辅助工具

#### Createhdds

```Createhdds``` 用于为某些 Rocky 测试准备 ```.img``` 和 ```.qcow2``` 文件。如果你运行了上述过程，你会注意到它在 ```/var/lib/openqa/factory/hdd/fixed``` 中生成了若干文件，这些文件由 [createhdds](https://github.com/rocky-linux/createhdds) 中提供的文件决定。

#### openqa-cli

测试通常使用 ```openqa-cli``` 发布，正如你上面已经使用过的那样。测试参数在 [openQA VARIABLES 定义文档](https://github.com/rocky-linux/os-autoinst-distri-rocky/blob/develop/VARIABLES.md) 中列出并解释。

#### 脚本

[辅助脚本](https://github.com/rocky-linux/os-autoinst-distri-rocky/tree/develop/scripts)——
当你发现你启动了一个大型构建但搞错了时，```cancel-build.sh``` 特别有用... 哎呀。

### 使用模板

#### 挑战

在测试操作系统时，特别是在进行持续测试时，出现的挑战之一是，每个修订版本总需要运行一组特定的作业组合，每个作业都有其自己的设置。这些组合对于同一修订版本的不同 ```FLAVOR``` 可能不同，比如为每种架构运行不同的作业集。如果 openQA 用于不同类型的测试，例如为某些快照运行简单的预集成测试，并结合为候选发布版本运行更全面的后集成测试，这种组合问题可能会进一步升级。

本节描述了如何*可能*使用 webUI 管理区域的选项配置一个 openQA 实例，以自动为每个需要测试的操作系统修订版本创建所有必需的作业。*如果*你从零开始（困难的方式），你可能会按以下顺序进行：

1. 在 'Machines'（机器）菜单中定义机器
2. 在 'Medium types'（介质类型）菜单中定义介质类型（产品）
3. 在 'Test suites'（测试套件）菜单中指定你要运行的各种测试集合
4. 在 'Job groups'（作业组）菜单中为测试组定义作业组
5. 选择各个 'Job groups'（作业组）并决定哪些组合是合理的且需要测试

如果你按照上面的安装指南操作，那么从 [os-autoinst-distri-rocky](https://github.com/rocky-linux/os-autoinst-distri-rocky) 克隆的 Rocky 测试将预配置 webUI 的管理区域。在阅读以下各节时，你可能会发现参考它很有用。

机器、介质、测试套件和作业模板都可以设置各种配置变量。作业组中的作业模板定义了测试套件、介质和机器如何以各种方式组合以产生单个"作业"。来自测试套件、介质、机器和作业模板的所有变量与作业创建请求中指定的变量一起被组合，并提供给由"作业"运行的实际测试代码。某些变量还会影响 openQA 和/或 os-autoinst 自身在如何为作业配置环境方面的行为。

配置从 ```/var/lib/openqa/tests/rocky/templates.fif.json``` 设置。

#### 机器

你至少需要设置一台机器才能运行任何测试。这些机器代表你想要测试的虚拟机类型。实际上，要使测试实际发生，你必须有一定数量的 ```openQA worker``` 连接，能够满足这些规格。

- ```Name```（名称）用户定义的 ```string```（字符串）——仅用于操作员识别机器配置。
- ```Backend```（后端）此 ```machine```（机器）应使用哪个 ```backend```（后端）。推荐值为 ```qemu```，因为它是测试最充分的，但也可以使用其他选项，如 ```kvm2usb``` 或 ```vbox```。
- ```Variables```（变量）大多数机器变量影响 os-autoinst 在设置测试机器方面的行为。一些重要的例子：
    - ```QEMUCPU``` 可以是 ```qemu32``` 或 ```qemu64```，指定虚拟 CPU 的架构
    - ```QEMUCPUS``` 是一个 ```integer```（整数），指定你希望使用的核心数

    - ```USBBOOT``` 当设置为 ```1``` 时，镜像将通过模拟的 USB 存储设备加载。

#### 介质类型

- ```product```（产品）
    - openQA 中的介质类型 ```product```（产品）是一个没有确定含义的简单描述。它基本上由一个 ```name```（名称）和一组 ```variables```（变量）组成，这些变量在 os-autoinst 中定义或描述此产品。

一些示例变量：

- ```ISO_MAXSIZE``` 包含 ```product```（产品）的最大大小。有一个测试检查产品的当前大小是否小于或等于此变量。
- ```DVD``` 如果设置为 ```1```，表示该介质是一个 DVD。
- ```LIVECD``` 如果设置为 ```1```，表示该介质是一个实时镜像（可以是 ```CD``` 或 ```USB```）
- ```GNOME``` 此变量如果设置为 ```1```，表示它是一个仅 ```GNOME``` 的发行版。
- ```RESCUECD``` 对于救援 CD 镜像设置为 ```1```。

#### 测试套件

一个测试套件由一个名称、一组在此特定测试内部使用的测试变量以及一个可选的描述组成。测试变量可用于参数化实际的测试代码并根据设置影响行为。

一些示例变量：

- ```DESKTOP```（桌面）可能的值为 ```kde``` ```gnome``` ```lxde``` ```xfce``` 或 ```textmode```。用于指示用户在测试期间选择的桌面。
- ```ENCRYPT```（加密）通过 ```YaST``` 加密主目录
- ```HDDSIZEGB``` 硬盘大小（GB）。
- ```HDD_1``` 预创建硬盘的路径
- ```RAIDLEVEL RAID``` 配置变量

#### 作业组

作业组是通过选择介质类型、测试套件和机器以及优先级值来定义实际测试场景的地方。

优先级值用于调度器选择下一个作业。如果多个作业被调度且运行它们的要求都得到满足，则优先级值较低的作业先被触发。ID 是具有相同要求和相同优先级值的两个作业的第二个排序键，ID 较低的先被触发。

作业组本身可以通过 Web UI 以及 REST API 创建。作业组可以选择性地嵌套到类别中。作业组和类别的显示顺序可以通过 Web UI 中的拖放来配置。

作业组中的场景定义可以通过不同的方式创建和配置：

- 一个简单的 Web UI 向导，当新的介质添加到作业组时会自动显示。
- Web UI 中的一个直观表格，用于向现有介质添加额外的测试场景，包括配置优先级值的可能性。
- 脚本 ```openQA-load-templates``` 和 ```openQA-dump-templates```，使用 REST API 从自定义的纯文本转储格式文件快速转储和加载配置。
- 使用 YAML 格式的声明式调度定义，通过 REST API 路由或 Web UI 中的在线编辑器（包括语法检查器）进行。

### Needles

Needles 非常精确，与指定显示的细微偏差都会被检测到。这意味着每次新版本发布时，显示的布局都会发生非常小的变化，导致需要大量新的或修改的 needles。测试团队在为新版本制作自动化测试时总是需要投入大量工作。

webui 的一个非常有用的功能是在线 needle 编辑器。当测试因缺少 needle 而失败时，可以通过点击图标激活 needle 编辑器，通常是通过复制一个相似的 needle 以及当前截图来创建一个新的 needle。needle 文件保存在 ```/var/lib/openqa/tests/rocky/needles``` 目录中。

### 上游文档

[入门指南](http://open.qa/docs/) 和 [上游文档](https://github.com/os-autoinst/openqa/blob/master/docs/Installing.asciidoc) 作为参考很有用，但由于它们混合了与 openSUSE 和 Fedora 相关的建议和说明，且两者之间存在实质性差异，因此并不总是清楚哪些对 Rocky 是重要的。然而，作为基于 rpm 的发行版，Rocky Linux 的使用与 [Fedora](https://fedoraproject.org/wiki/OpenQA) 版本有松散的关联。

### 术语表

以下术语在 openQA 上下文中使用：-

- test module（测试模块）
    - 单个 Perl 模块 ```.pm``` 文件中的单个测试用例，例如 ```sshxterm```。如果没有进一步指定，测试模块由其短 ```name```（名称）表示，等同于包含测试定义的文件名。完整 ```name```（名称）由测试组（由测试模块文件的顶层文件夹形成）和短名称组成，例如 ```x11-sshxterm```（对应 ```x11/sshxterm.pm```）

- test suite（测试套件）
    - 一组测试模块，例如 ```textmode```。一个测试套件中的所有测试模块按顺序运行

    - 由一个 openQA 实例的唯一编号表示的一组单个测试用例的运行，例如在 ```gnome``` 内进行一次安装，随后测试应用程序

- test run（测试运行）
    - 等同于 job（作业）
- test result（测试结果）
    - 一个作业的结果，例如 ```passed```（通过），以及每个测试模块的详细信息
- test step（测试步骤）
    - 在作业内执行一个测试模块

- distri
    - 测试发行版，但有时也指 ```product```（产品）（注意：有歧义，历史上是"GNU/Linux 发行版"），由多个测试模块组成文件夹结构，构成测试套件，例如 ```rocky```（测试发行版，是 ```os-autoinst-distri-rocky``` 的缩写）

- product（产品）
    - 主要的 ```system under test (SUT)```（被测系统）例如 ```rocky```，在 openQA 的 Web 界面中也称为 ```Medium Types```（介质类型）

- job group（作业组）
    - 等同于 product（产品），在 webUI 上下文中使用

- version（版本）
    - 产品的一个版本，不要与 ```build```（构建）混淆

- flavor
    - 产品特定变体的关键字，用于区分不同的变体，例如 ```dvd-iso```

- arch
    - 产品的架构变体，例如 ```x86_64```

- machine（机器）
    - 设备的额外变体，例如用于 ```64bit``` ```bios``` ```uefi``` 等。

- scenario（场景）
    - 一个组合：```<distri>-<version>-<flavor>-<arch>-<test_suite>@<machine>``` 例如 ```Rocky-9-dvd-x86_64-gnome@64bit```

- build（构建）
    - 待测试产品的不同版本，可视为 ```version```（版本）的 ```sub-version```（子版本），例如 ```Build1234``` 注意：歧义：前缀 ```build``` 可能包含或不包含

### 历史（简述）

openQA 起源于 OS-autoinst：操作系统的自动化测试。OS-autoinst 项目旨在提供一种运行全自动化测试的手段，尤其是对操作系统基本和底层组件（如引导加载程序、内核、安装器和升级）进行测试，这些组件不容易用其他自动化测试框架进行测试。然而，它同样可以用于测试在新安装的操作系统上运行的 firefox 和 openoffice 的操作。

openQA 是一个测试调度器和 Web 前端，使用 OS-autoinst 作为后端，用于 openSUSE 和 Fedora。

openQA 起源于 openSUSE，并被 Fedora 采用作为其频繁发行版更新的自动化测试系统。维护活动相当密集，并在各层次的用户中持续进行。openQA 被 Rocky Linux 测试团队采用作为其发行版持续发布的优选自动化测试系统。

openQA 是在 GPLv2 许可下发布的自由软件。

### 署名

本指南深受描述 OS-autoinst 和 openQA 安装和使用的众多上游文档的启发。

### 参考资料

由于 Rocky Linux 对 openQA 的使用源自上游 Fedora，进而源自 openSUSE，本文档包含**许多**来自上游文档的编辑过的段落，在此衷心感谢这些内容的使用。与许多开源项目一样，我们是站在前人的工作基础上构建的。

另请参见：[安装信息](https://github.com/rocky-linux/OpenQA-Fedora-Installation) 了解更多详情。

### 修订历史

- v1.3 - 2026/04/28 - 修复失效 URL 并修改 curl 以防止重复
- v1.2 - 2026/04/16 - 添加 content_bottom.md include
- v1.1 - 2025/06/05 - 次要更新
- v1.0 - 2024/04/30 - 首次发布

{% include 'teams/testing/content_bottom.md' %}
