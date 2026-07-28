---
title: Sparky 测试入门
author: Chris "Stack" Stackpole
contributors: Alexey Melezhik, Howard Van Der Wal
ai_contributors: Claude (claude-opus-4-8)
tested_with: 9.8
tags:
  - automation
  - raku
  - sparky
  - testing
---

**知识量**: :star:  
**阅读时间**: 30 分钟

## AI 使用

本文档遵循[此处提供的 AI 贡献政策](../contribute/ai-contribution-policy.md)。如果您发现说明中有任何错误，请告知我们。

## 这是什么以及为什么？

本指南介绍如何安装、配置和运行 Sparky，以便您能够针对 Rocky Linux 运行 Rocky Linux 测试团队的自动化用例测试。

### Rocky Linux 测试核心组件简介

Perl 是一种编程语言。Raku^2^ 是 Perl 家族的一员（即 Perl 6）。Sparrow^6^ 是一个用 Raku 构建的自动化框架。Sparky^4^ 是一个用 Raku 编写的持续集成服务器和分布式任务运行器，它使用 Sparrow。

### 为什么选择 Sparky 和 Sparrow 用于 Rocky Linux 测试？

测试的目标是获得更多的常见用例测试覆盖率，以便 Rocky 测试团队在新的 Rocky Linux 发布版（主要版和修订版）上进行验证。此外，还有两个其他目标：

1. 由于测试可以是相当简单的 Bash 代码，我们希望更多的系统管理员能够为他们关心的事项编写自动化测试。这为 Rocky Linux 社区成员贡献测试提供了绝佳的机会。
2. 测试可用于验证文档。这为文档团队提供了机会，可以在用户发现之前（例如）发现文档中的软件包损坏问题。

OpenQA、Kickstart 测试和 Sparky 的维恩图重叠度相当小。虽然测试团队曾希望将大部分文档集成到 OpenQA 或 Kickstart 测试中，但这个过程变得复杂。Sparky 提供了一个自动化框架工具，Rocky Linux 社区更容易为此做出贡献。

## 前置条件

- 在 x86_64 或 aarch64 上安装 Rocky Linux 9 基础系统。RISC-V 说明假设使用 Rocky Linux 10。
- 通过 `sudo` 获得管理权限以安装软件包和打开防火墙端口。
- 可选：您可以在虚拟机内运行此程序，只要宿主机提供 QEMU 透传以支持嵌套虚拟化。此设置不在本指南范围内，但如果需要进行大量配置测试且需要快速重启或重置，这会有所帮助。

## 安装

本指南应包含您使用 Sparky 运行 Rocky 测试脚本所需的一切。

### 安装依赖项

```bash
sudo dnf install -y wget tar perl xorriso bash-completion qemu-kvm vim sqlite git openssl-devel rsync tmux genisoimage
```

### 为 x86_64 和 aarch64 安装 Raku

有两种方式，推荐的方式是从软件包仓库安装：

#### 从软件包仓库安装

```bash
sudo rpm --import 'https://dl.cloudsmith.io/public/nxadm-pkgs/rakudo-pkg/gpg.0DD4CA7EB1C6CC6B.key'
sudo curl -1sLf 'https://dl.cloudsmith.io/public/nxadm-pkgs/rakudo-pkg/config.rpm.txt?distro=el&codename=9' > /tmp/nxadm-pkgs-rakudo-pkg.repo
sudo dnf config-manager --add-repo '/tmp/nxadm-pkgs-rakudo-pkg.repo'
sudo dnf --refresh -y install rakudo-pkg
# rakudo 和 zef 都需要添加到 PATH 中
/opt/rakudo-pkg/bin/add-rakudo-to-path
/opt/rakudo-pkg/bin/install-zef
# 确保当前环境匹配
source ~/.bash_profile
# 验证 raku 在 PATH 中
which raku
# 验证 zef 在 PATH 中
which zef
```
#### 下载并运行远程脚本

```bash
curl https://rakubrew.org/install-on-perl.sh | sh
echo 'eval "$(~/.rakubrew/bin/rakubrew init Bash)"' >> ~/.bashrc
eval "$(~/.rakubrew/bin/rakubrew init Bash)"
# 验证最新版本
rakubrew download moar-2026.05
```

### 在 RISC-V 上安装 Raku

RISC-V 目前尚无软件包仓库，因此目前需要手动编译。此外，还需要为该架构配置一些更改。

!!! warning

    RISC-V 支持仍处于早期测试阶段。

```bash
sudo dnf -y group install "Development Tools"
sudo dnf -y install libffi-devel
curl https://rakubrew.org/install-on-perl.sh | sh
echo 'eval "$(~/.rakubrew/bin/rakubrew init Bash)"' >> ~/.bashrc
eval "$(~/.rakubrew/bin/rakubrew init Bash)"
# 在此阶段几乎肯定会因为 MoarVM 和 dyncall 而失败。后续步骤将修复它
rakubrew build moar-2026.02
cd ~/.rakubrew/versions/moar-2026.02/nqp/MoarVM/
/usr/bin/perl Configure.pl --optimize --git-cache-dir=~/.rakubrew/git_reference --prefix=~/.rakubrew/versions/moar-2025.08/install --make-install --has-libffi
# 这应该会找到一个可用的 MoarVM 并继续编译
rakubrew build moar-2026.02
rakubrew build zef
```

现在测试是否正常工作：

```bash
raku -v
```

## 克隆 Sparky

此 `zef install .` 命令很可能会失败某个测试。

如果错误是缺少包，请尝试安装缺少的包。

上面的安装应该涵盖了所有需要的依赖项，但例如包的可用性可能会发生变化。

您可能需要运行 `dnf provides */missing.so` 搜索。如果错误是关于失败的测试，那么尝试运行以下命令：

```bash
zef install --/test .
```

由于一个已知问题，下面概述的三阶段安装过程是最可靠的：

```bash
mkdir ~/Code
cd ~/Code
git clone https://github.com/melezhik/sparky.git
cd sparky
zef install DBIish --/test
zef install cro --deps-only
zef install cro
zef install .
```

### 安装 Sparky

仅在 `zef install .` 完成后运行此命令：

```bash
raku db-init.raku
```

但是请注意——有时您可能会收到类似下面描述的错误：

`⚠ sparky DBDish::SQLite: Error: no such table: builds (1)`

这可能是由于损坏的数据库导致（例如，您从另一个用户的仓库复制了数据库，或者某个测试出了严重问题）。该命令将重置数据库。

### 安装 Sparky Job API

运行以下命令：

```bash
zef install Sparky::JobApi
```

如果成功，您将看到以下输出：

```bash
All candidates are currently installed
```

## 配置

### 防火墙

确保防火墙允许端口 4000：

```bash
sudo firewall-cmd --add-port=4000/tcp --permanent
sudo firewall-cmd --reload
```

## 项目设置

现在是时候加载一些项目了。

### 代码仓库准备

```bash
cd ~/Code
git clone https://git.resf.org/testing/Sparky_Rocky
cd Sparky_Rocky
bash scripts/sync_project.sh
```

预期输出如下：

```text
Creating project folder in ~/.sparky
Checking for ~/sparky.yaml
Creating API Key in ~/sparky.yaml
Checking for ~/.sparky/templates/vars.yaml
Copying project files
```

#### 关于 API key 的说明

这可以是并且应该是一个随机字符串。这需要放在安装和运行 Sparky 服务器的任何系统上——在本演示中到目前为止，就是您的宿主机。此 key 应该已由上述脚本生成，并且可以在 `~/sparky.yaml` 中找到。

### 下载镜像

无需手动下载镜像。可用镜像在您的 `~/.sparky/templates/vars.yaml` 文件中的 `qemu_image` 列表中定义，当作业运行时，Sparky 会下载您选择的那个。默认情况下，该列表附带 Rocky Linux 9 x86_64、Rocky Linux 10 x86_64 和 Rocky Linux 10 aarch64。

要添加其他镜像，或更改现有镜像的下载 URL，编辑该列表（如果您不确定如何使用 `nano` 编辑器，请查看此 man 页面^1^）：

```bash
nano ~/.sparky/templates/vars.yaml
```

在 `vars` -> `qemu_test` 下，每个 `qemu_image` 条目是一个名称和一个下载 URL，用冒号连接（`名称:URL`）：

```yaml
vars:
  qemu_test:
    qemu_image:
      - "Rocky-9-GenericCloud:https://dl.rockylinux.org/pub/rocky/9/images/x86_64/Rocky-9-GenericCloud-Base.latest.x86_64.qcow2"
      - "Rocky-10-GenericCloud:https://dl.rockylinux.org/pub/rocky/10/images/x86_64/Rocky-10-GenericCloud-Base.latest.x86_64.qcow2"
      - "Rocky-10-GenericCloud-Aarch:https://dl.rockylinux.org/pub/rocky/10/images/aarch64/Rocky-10-GenericCloud-Base.latest.aarch64.qcow2"
```

例如，要添加 Rocky Linux 8 镜像，追加另一行：

```yaml
      - "Rocky-8-GenericCloud:https://dl.rockylinux.org/pub/rocky/8/images/x86_64/Rocky-8-GenericCloud-Base.latest.x86_64.qcow2"
```

刷新 Web 界面，新镜像将出现在工作页面的 `version` 下拉菜单中。选择它并使用 `bootstrap` 复选框启动一个作业。Sparky 首次会下载镜像，之后会重用缓存的副本。

### 配置 Sparky

`vars.yaml` 文件包含许多重要的变量。模板应该已由上述 `sync_project.sh` 脚本创建。请验证您的 `~/.sparky/templates/vars.yaml` 文件。模板可以在 Sparky_Rocky^5^ 项目的 examples 文件夹下找到。

#### 验证 QEMU 二进制文件

您的 QEMU 二进制文件可能不同。您可能还需要指定机器类型。这是一个帮助您找到 QEMU 二进制文件的简单指南，但并非详尽列表。一旦您知道了二进制文件/路径——就在 `vars.yaml` 文件中更新它。

```bash
which qemu-kvm
ls -lh /usr/libexec/qemu*
# 如果 QEMU 二进制文件不在您的 PATH 中，这里有几种确保它在 PATH 中的方法。逐一尝试，直到找到有效的方法。
alias qemu-kvm=/usr/libexec/qemu-kvm
echo 'alias qemu-kvm=/usr/libexec/qemu-kvm' >> ~/.bash_profile
mkdir ~/bin && ln -s /usr/libexec/qemu-kvm ~/bin/qemu-kvm
```

#### 验证 QEMU 机器

找到 QEMU 二进制文件后，您需要知道系统支持哪些机器类型。从以下命令生成的列表中选择最佳的一个：

```bash
qemu-kvm -machine help
```

以下是来自 Rocky Linux 9 x86_64 机器的示例输出：

```bash
[howard@rocky9 ~]$ qemu-kvm -machine help
Supported machines are:
pc                   RHEL 7.6.0 PC (i440FX + PIIX, 1996) (alias of pc-i440fx-rhel7.6.0)
pc-i440fx-rhel7.6.0  RHEL 7.6.0 PC (i440FX + PIIX, 1996) (default) (deprecated)
q35                  RHEL-9.8.0 PC (Q35 + ICH9, 2009) (alias of pc-q35-rhel9.8.0)
pc-q35-rhel9.8.0     RHEL-9.8.0 PC (Q35 + ICH9, 2009)
pc-q35-rhel9.6.0     RHEL-9.6.0 PC (Q35 + ICH9, 2009)
pc-q35-rhel9.4.0     RHEL-9.4.0 PC (Q35 + ICH9, 2009)
pc-q35-rhel9.2.0     RHEL-9.2.0 PC (Q35 + ICH9, 2009)
pc-q35-rhel9.0.0     RHEL-9.0.0 PC (Q35 + ICH9, 2009)
pc-q35-rhel8.6.0     RHEL-8.6.0 PC (Q35 + ICH9, 2009) (deprecated)
pc-q35-rhel8.5.0     RHEL-8.5.0 PC (Q35 + ICH9, 2009) (deprecated)
pc-q35-rhel8.4.0     RHEL-8.4.0 PC (Q35 + ICH9, 2009) (deprecated)
pc-q35-rhel8.3.0     RHEL-8.3.0 PC (Q35 + ICH9, 2009) (deprecated)
pc-q35-rhel8.2.0     RHEL-8.2.0 PC (Q35 + ICH9, 2009) (deprecated)
pc-q35-rhel8.1.0     RHEL-8.1.0 PC (Q35 + ICH9, 2009) (deprecated)
pc-q35-rhel8.0.0     RHEL-8.0.0 PC (Q35 + ICH9, 2009) (deprecated)
pc-q35-rhel7.6.0     RHEL-7.6.0 PC (Q35 + ICH9, 2009) (deprecated)
none                 empty machine
```

然后在文本编辑器中打开 `~/.sparky/templates/vars.yaml` 配置。

您将看到类似如下的代码块：

```yaml
    qemu:
      binary: qemu-kvm
      #binary: qemu-system-x86_64
      machine: ""
      #machine: pc-q35-rhel9.4.0
```

从上面的示例中，您可以看到它没有使用最新的机器设置 `pc-q35-rhel9.8.0`。

在这种情况下，您需要像这样设置 `vars.yaml` 配置：

```yaml
    qemu:
      binary: qemu-kvm
      #binary: qemu-system-x86_64
      machine: "pc-q35-rhel9.8.0"
      #machine: pc-q35-rhel9.4.0
```

#### 验证 SSH

如果您没有 SSH key，运行以下命令：

```bash
ssh-keygen -t ed25519
```

按 "enter" 接受所有默认值。

然后使用 `ls -l ~/.ssh/` 验证您的 key 路径。输出类似如下：

```bash
[howard@rocky9 ~]$ ls -l ~/.ssh/
total 8
-rw-------. 1 howard howard 399 Jun 15 13:46 id_ed25519
-rw-r--r--. 1 howard howard  95 Jun 15 13:46 id_ed25519.pub
```

然后将您新创建的 SSH 公钥添加到 `vars.yaml` 文件的底部。如下所示：

```yaml
    ssh_key_path:
      # 可以有多个 key
      # - ~/.ssh/foo.bar.pub
      - ~/.ssh/id_rsa.pub
      - ~/.ssh/id_ed25519.pub
```

有关 SSH 密钥的更多信息，请查看 Rocky Linux SSH 指南^7^。

#### 验证 sudo 权限

对于 kickstart 测试，需要 sudo 无密码挂载才能通过测试。

运行以下命令编辑 `sudoers` 文件：

```bash
sudo visudo
```

然后添加此行（可以放在文件中的任何位置），将 `YOUR_USER_NAME_HERE` 替换为您的用户名：

```bash
## Sparky users
YOUR_USER_NAME_HERE ALL=(ALL) NOPASSWD: /usr/bin/mount,/usr/bin/umount
```

默认情况下，`visudo` 使用 `vi` 文本编辑器打开 `sudoers` 文件。有关 `vi` 的良好入门介绍，请阅读 Rocky Linux 关于 `vi` 文本编辑器的书籍^9^。

## 启动服务

现在，有几种方法可以执行下一步——`tmux`、`screen`、打开多个 shell，或者只是将任务后台运行。哪种方式都可以。在调试时看到消息很有帮助，因此您可能希望打开新的 shell 来观察它们。

### 分窗口/标签页方法

在单独的窗口/标签页/后台中运行这两个命令。它们应从 `sparky` 目录 `~/Code/sparky` 运行：

```bash
# 窗口/标签页 1
sparkyd
```

```bash
# 窗口/标签页 2
cro run
```

### tmux 方法

有关导航和操作 `tmux` 的更多信息，请查看 tmux 速查表^8^：

```text
tmux
sparkyd
"Ctrl+b" 然后 ":new"
cd ~/Code/sparky
cro run
"Ctrl+b" 然后 "s" 切换会话
"Ctrl+b" 然后 "d" 分离
# 列出所有 tmux 会话：
tmux a
# 重新附加到某个会话：
tmux a -t <SESSION_NUMBER>
```

如果您收到类似 "Did not encounter any `.cro.yml` files... could it be `.cro.yml` file is missing?" 的错误，在运行上述命令之前，请确保您在 sparky 文件夹 `cd ~/Code/sparky` 中并再次尝试 `cro run`。如果正常工作，您应该会看到 `sparky run sparky web ui on host: 0.0.0.0, port: 4000` 的消息。

### 启动 Web 浏览器

现在在 Web 浏览器中打开 `http://your-IP:4000`

以 `admin`:`admin` 登录

### 启动测试

现在转到 `Projects`。您应该看到 "sparky-rocky"。

点击链接查看项目详情，然后点击 "Build now"。

然后您将有三个选项：`qemu`、`qemu_with_kickstart` 和 `orb`。选择 `qemu`。

可用选项：

#### version

要测试的镜像。此下拉菜单由 `~/.sparky/templates/vars.yaml` 中的 `qemu_image` 列表填充，因此它会显示您配置的镜像名称（例如 `Rocky-9-GenericCloud`）。

#### releasever

选择与您的镜像匹配的版本。本指南中使用的示例是 Rocky Linux 9.8，因此这里选择 `9.8`。

#### use_case_repo

您要对正在部署的镜像运行哪些测试。对于测试运行，您可以选择 Sparky_WP_LAMP^3^ 测试。

#### qemu_binary

默认情况下，只有 `qemu-kvm` 可选，因此保持此选项不变。

#### qemu_machine

这将预填充您在 `~/.sparky/templates/vars.yaml` 中设置的 `machine`。在本示例中，它是 `pc-q35-rhel9.8.0`。

#### ssh_key_path

这将同样预填充您在 `~/.sparky/templates/vars.yaml` 中的 SSH key 创建步骤中设置的 key。

#### bootstrap

如果选中此选项，镜像将从头开始重建，并且 `Sparrow` 客户端将在镜像上全新安装。

#### qemu_shut

选中此选项意味着测试完成后，Sparky 将关闭 QEMU 会话。不选中此选项则保留 QEMU 会话运行，以防您之后需要访问该会话。

#### skip_test

在配置 VM 时，选中此选项不会运行实际的用例测试。这在您只想要一个干净运行的 VM（或准备镜像）而不需要测试阶段时很有用。

#### dump_task_code

这会在每个任务运行时打印其源代码，用于需要调试 Sparky 的场景。它是 Sparrow6 框架的一部分。

### 首次运行

初次运行时，请确保选中 bootstrap 并点击 "Submit"。

您应该在启动 `sparkyd` 和 `cro run` 的两个窗口窗格中看到活动。

您还应看到 "build has started"，它将变为一个作业编号链接，指向作业报告。

点击 "Recent Builds"。您应该看到构建正在运行。点击 ID 链接查看作业处于什么状态的详细信息。

当 QEMU 会话首次启动时，将需要几分钟时间来运行。

### 关闭会话

对于 QEMU 会话未关闭的情况，无论是由于上述选项之一还是未能关闭它的故障，运行以下命令：

```bash
ssh -p 10022 admin@127.0.0.1
sudo init 0
```

如果有关于不良 ssh 密钥的通知，请使用以下命令从 `~/.ssh/known_hosts` 中单独删除有问题的条目，或者如果不需要，可以删除整个 `known_hosts` 文件：

```bash
ssh-keygen -R [127.0.0.1]:10022
```

端口号会根据启动的 QEMU 实例数量而变化。

您应该能够在测试报告中观察到端口号。

### 干净会话开始

有些测试不会影响其他测试，而其他测试需要新的 QEMU VM 镜像。

默认情况下，相同的 QEMU 镜像将被重用。

如果您希望全新安装镜像，请通过作业 UI 启用 `bootstrap` 选项。

在运行任何测试时，您应该始终能够以全新的 QEMU 镜像开始。

如果您在重用镜像时遇到失败，请在提交报告之前使用全新的 QEMU 镜像验证问题。

## 结果确认

在 `Recent Builds` 下，点击每个结果的链接，在 `Report` 或 `System Log` 下，将告诉您作业是否成功。

### Rocky Linux LAMP 检查的结果

将启动几个子作业。Rocky 8.10 和 9.8 的结果应该相同：

- sparky-rocky - 由 admin 触发 - 应以 "succeed" 结果结束
- qemu-session - 启动 qemu 会话 - 应处于 "running" 状态
- qemu-use-case - 用例场景 - 应以 "succeed" 结果结束

### ZFS 测试的结果

将启动几个子作业。结果适用于 Rocky 9.8，因为 8.10 需要的调整尚未包含在 ZFS 测试中。

- sparky-rocky - 由 admin 触发 - 应以 "succeed" 结果结束
- qemu-session - 启动 qemu 会话 - 应处于 "running" 状态
- qemu-use-case - 用例场景 - 应以 "succeed" 结果结束
- qemu-reboot - 重启 qemu 机器 - 应以 "succeed" 结果结束
- qemu-use-case - 用例场景 - 应以 "succeed" 结果结束

点击每个作业的 ID 获取更多详细信息。您应该看到 `zpool status`、`zpool iostat` 和 `zpool list` 的成功横幅。验证在 "[task check]" 部分下所有项目都返回了 "True"。

### Slurm 测试的结果

将启动几个子作业。结果适用于 Rocky 9.8，因为 8.10 在此时安装 Slurm 存在问题。

- sparky-rocky - 由 admin 触发 - 应以 "succeed" 结果结束
- qemu-session - 启动 qemu 会话 - 应处于 "running" 状态
- qemu-use-case - 用例场景 - 应以 "succeed" 结果结束

点击作业的 ID 获取更多详细信息。您应该看到成功分区（"normal*      up 7-00:00:00      1   idle master"），随后是 "Test complete!" 的消息。

## 编写新测试

### 创建仓库

对于此步骤，您将运行多个 `git` 命令。如果您不确定如何使用 `git`，请查看 [Rocky Linux Gemstone for git](../../gemstones/git/01-gh_cli_1st_pr.md) 了解如何添加文件和提交更改。

首先，创建一个新目录来工作。此示例将使用 `sparky-newtest`，这将创建您需要的模板：

```bash
cd ~/Code
mkdir -p sparky-newtest/tasks/check-newtest
cd sparky-newtest
echo "This is your README content; please explain something about this test." > README.md
echo -e '#!raku\ntask-run "tasks/check-newtest";' > main.raku
echo -e '#!/bin/bash -\necho "new test code here"' > tasks/check-newtest/task.bash
```

`main.raku` 是告诉 Sparky 作业要运行哪些任务详情的文件。这是一个非常简单的示例；请参考文档了解更多详情。

`task.bash` 是要运行的任务。这是您要在 QEMU VM 内运行的 shell 脚本。

要将更改提交到仓库，以下是您可以运行的示例命令：

```bash
git init
git add .
git commit -m "Add sparky-newtest test scaffold"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```

### 修改 vars.yaml 查看您的测试

将代码提交到远程仓库后，编辑 `vars.yaml`：

```bash
nano ~/.sparky/templates/vars.yaml
```

在 `vars` -> `qemu_test` -> `use_case_repo` 下，添加一个遵循 YAML 语法的新条目和 Git URL。示例配置如下：

```yaml
    use_case_repo:
      - https://github.com/metalllinux/Sparky_Test
      - https://git.resf.org/testing/Sparky_Bind
```

刷新 Web 界面。它应该出现在 `use_case_repo` 列表的底部。

一旦有可以正常工作的内容，提交 Pull Request 让我们添加该仓库。

## 升级

要升级，在宿主机上的任何位置重新运行 `zef` 命令：

```bash
zef upgrade --/test Sparrowdo
```

如果您的 `Sparrowdo` 已是最新版本，您将看到以下输出：

```text
===> Searching for: Sparrowdo
All requested distributions are already at their latest versions
```

## 总结

有了 Sparky 安装、配置和运行，您可以针对 Rocky Linux 镜像启动 Rocky 测试脚本，并在 Web 界面中查看结果。

从那里，您可以编写自己的测试并提交给 Rocky Linux 测试团队，让更广泛的社区从测试覆盖中受益。

## 参考资料

1. "nano man pages" by rockyman.org [https://rockyman.org/9.8/nano/man1/nano.1.html](https://rockyman.org/9.8/nano/man1/nano.1.html)
2. "Raku" by The Raku Project [https://raku.org/](https://raku.org/)
3. "Rocky Linux LAMP Check" by Alexey Melezhik [https://github.com/melezhik/rocky-linux-lamp-check](https://github.com/melezhik/rocky-linux-lamp-check)
4. "Sparky Project" by Alexey Melezhik [https://github.com/melezhik/sparky](https://github.com/melezhik/sparky)
5. "Sparky Rocky" by Alexey Melezhik and Chris "Stack" Stackpole [https://git.resf.org/testing/Sparky_Rocky](https://git.resf.org/testing/Sparky_Rocky)
6. "Sparrow" by Alexey Melezhik [https://github.com/melezhik/Sparrow6](https://github.com/melezhik/Sparrow6)
7. "SSH public and private key" by Steven Spencer, Ezequiel Bruni, and Ganna Zhyrnova [https://docs.rockylinux.org/guides/security/ssh_public_private_keys/?h=ssh+key](https://docs.rockylinux.org/guides/security/ssh_public_private_keys/?h=ssh+key)
8. "Tmux Cheat Sheet & Quick Reference" by l9c and contributors [https://tmuxcheatsheet.com/](https://tmuxcheatsheet.com/)
9. "VI Text Editor" by Antoine Le Morvan, Ezequiel Bruni, Ganna Zhyrnova, Patrick Starrenburg, Serge Croisé, and Tianci Li [https://docs.rockylinux.org/10/books/admin_guide/05-vi/](https://docs.rockylinux.org/10/books/admin_guide/05-vi/)
