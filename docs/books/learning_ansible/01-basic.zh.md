---
title: Ansible 基础
author: Antoine Le Morvan
contributors: Steven Spencer, tianci li, Aditya Putta, Ganna Zhyrnova
update: 15-Dec-2021
---

# Ansible 基础

在本章中，你将学习如何使用 Ansible。

****

**目标**：在本章中你将学习：

:heavy_check_mark: 实现 Ansible；
:heavy_check_mark: 在服务器上应用配置更改；
:heavy_check_mark: 创建第一个 Ansible playbook（剧本）；

:checkered_flag: **ansible**、**module**、**playbook**。

**知识**：:star: :star: :star:
**复杂度**：:star: :star:

**阅读时间**：30 分钟

****

Ansible 集中化和自动化管理任务。它是：

* **无代理**（agentless）的（不需要在客户端上进行特定部署），
* **幂等**（idempotent）的（每次运行时产生相同的效果）。

它使用 **SSH** 协议来远程配置 Linux 客户端，或使用 **WinRM** 协议来处理 Windows 客户端。如果这些协议都不可用，Ansible 始终可以使用 API，这使得 Ansible 成为配置服务器、工作站、docker 服务、网络设备等（几乎是所有事物）的真正瑞士军刀。

!!! Warning

    从 Ansible 服务器向所有客户端开放 SSH 或 WinRM 流，使其成为架构中的关键元素，必须仔细监控。

由于 Ansible 主要是基于推送（push-based）的，它不会在每次执行之间保持其目标服务器的状态。相反，它将在每次执行时执行新的状态检查。它被称为无状态的（stateless）。

它将帮助你完成：

* 资源调配（deploying a new VM），
* 应用程序部署，
* 配置管理，
* 自动化，
* 编排（当使用超过 1 个目标时）。

!!! Note

    Ansible 最初由 Michael DeHaan 编写，他是其他工具如 Cobbler 的创始人。

    ![Michael DeHaan](images/Michael_DeHaan01.jpg)

    最早的版本是 0.0.1，于 2012 年 3 月 9 日发布。

    2015 年 10 月 17 日，AnsibleWorks（Ansible 背后的公司）被 Red Hat 以 1.5 亿美元收购。

![Ansible 的功能特性](images/ansible-001.png)

要为你的日常 Ansible 使用提供图形界面，你可以安装一些工具，如 Ansible Tower（RedHat，非免费）、其开源对应物 Awx，或其他项目如 Jenkins 和优秀的 Rundeck 也可以使用。

!!! Abstract

    要跟随此培训，你至少需要 2 台运行 Rocky8 的服务器：

    * 第一台将是**管理机**（management machine），Ansible 将安装在此机器上。
    * 第二台将是用于配置和管理的服务器（其他 Linux 发行版也可以，不仅仅是 Rocky Linux）。

    在下面的示例中，管理站的 IP 地址为 172.16.1.10，被管理站的 IP 地址为 172.16.1.11。你需要根据你的 IP 寻址计划调整示例。

## Ansible 术语表

* **管理机**（management machine）：安装了 Ansible 的机器。由于 Ansible 是**无代理**的，因此不会在被管理的服务器上部署任何软件。
* **被管理节点**（managed nodes）：Ansible 管理的目标设备，也称为 "hosts"。这些可以是服务器、网络设备或任何其他计算机。
* **清单**（inventory）：一个包含被管理服务器信息的文件。
* **任务**（tasks）：任务是定义要执行的过程的块（例如，创建用户或组，安装软件包等）。
* **模块**（module）：模块抽象了一个任务。Ansible 提供了许多模块。
* **Playbook**（剧本）：一个简单的 yaml 格式文件，定义了目标服务器和要执行的任务。
* **角色**（role）：角色允许你组织 playbook 和所有其他必要的文件（模板、脚本等），以便于共享和重用代码。
* **集合**（collection）：集合包括一组逻辑的 playbook、角色、模块和插件。
* **事实**（facts）：这些是包含系统信息的全局变量（机器名、系统版本、网络接口和配置等）。
* **处理器**（handlers）：这些用于在发生更改时停止或重启服务。

## 在管理服务器上安装

Ansible 在 _EPEL_ 仓库中可用，但有时版本对于当前版本来说可能太旧，你可能想要使用更新的版本。

因此，我们将考虑两种安装方式：

* 基于 EPEL 仓库的安装
* 基于 `pip` python 包管理器的安装

_EPEL_ 是两种版本都需要的，所以你现在可以开始安装它：

* EPEL 安装：

```bash
sudo dnf install epel-release
```

### 从 EPEL 安装

如果我们从 _EPEL_ 安装 Ansible，我们可以执行以下操作：

```bash
sudo dnf install ansible
```

然后验证安装：

```bash
$ ansible --version
ansible [core 2.14.2]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/rocky/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.11/site-packages/ansible  ansible collection location = /home/rocky/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.11.2 (main, Jun 22 2023, 04:35:24) [GCC 8.5.0 20210514 
(Red Hat 8.5.0-18)] (/usr/bin/python3.11)
  jinja version = 3.1.2
  libyaml = True

$ python3 --version
Python 3.6.8
```

请注意，ansible 带有自己版本的 python，与系统版本的 python 不同（此处为 3.11.2 与 3.6.8）。你需要在你通过 pip 安装你的安装所需的 python 模块时考虑到这一点（例如 `pip3.11 install PyVMomi`）。

### 从 python pip 安装

由于我们想要使用更新版本的 Ansible，我们将从 `python3-pip` 安装它：

!!! Note

    如果你之前从 _EPEL_ 安装了 Ansible，请先移除它。

在此阶段，我们可以选择使用我们想要的 python 版本来安装 ansible。

```bash
sudo dnf install python38 python38-pip python38-wheel python3-argcomplete rust cargo curl
```

!!! Note

    `python3-argcomplete` 由 _EPEL_ 提供。如果尚未安装，请安装 epel-release。
    此包将帮助你补全 Ansible 命令。

现在我们可以安装 Ansible：

```bash
pip3.8 install --user ansible
activate-global-python-argcomplete --user
```

检查你的 Ansible 版本：

```bash
$ ansible --version
ansible [core 2.13.11]
  config file = None
  configured module search path = ['/home/rocky/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/rocky/.local/lib/python3.8/site-packages/ansible
  ansible collection location = /home/rocky/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/rocky/.local/bin/ansible
  python version = 3.8.16 (default, Jun 25 2023, 05:53:51) [GCC 8.5.0 20210514 (Red Hat 8.5.0-18)]
  jinja version = 3.1.2
  libyaml = True
```

!!! NOTE

    我们手动安装的版本比 RPM 打包的版本更旧，因为我们使用了较旧的 python 版本。这个观察结果会随着时间和发行版以及 python 版本的不同而变化。

## 配置文件

服务器配置位于 `/etc/ansible` 下。

有两个主要的配置文件：

* 主配置文件 `ansible.cfg`，其中包含命令、模块、插件和 ssh 配置；
* 客户端机器管理清单文件 `hosts`，其中声明了客户端和客户端组。

如果 Ansible 通过其 RPM 包安装，配置文件将自动创建。通过 `pip` 安装时，此文件不存在。我们必须通过 `ansible-config` 命令手动创建：

```bash
$ ansible-config -h
usage: ansible-config [-h] [--version] [-v] {list,dump,view,init} ...

View ansible configuration.

positional arguments:
  {list,dump,view,init}
    list                Print all config options
    dump                Dump configuration
    view                View configuration file
    init                Create initial configuration
```

示例：

```bash
ansible-config init --disabled > /etc/ansible/ansible.cfg
```

`--disabled` 选项允许你通过在选项前添加 `;` 来注释掉整个选项集。

!!! NOTE

    你也可以选择将 ansible 配置嵌入到你的代码仓库中，Ansible 会按照以下顺序加载它找到的配置文件（处理找到的第一个文件并忽略其余的）：

    * 如果设置了环境变量 `$ANSIBLE_CONFIG`，则加载指定的文件。
    * 当前目录中的 `ansible.cfg`（如果存在）。
    * 用户家目录中的 `~/.ansible.cfg`（如果存在）。

    如果这三个文件都未找到，则加载默认文件。

### 清单文件 `/etc/ansible/hosts`

由于 Ansible 需要与你所有待配置的设备一起工作，因此为其提供一个（或多个）结构良好、完全符合你组织要求的清单文件是至关重要的。

有时需要仔细考虑如何构建此文件。

转到默认的清单文件，该文件位于 `/etc/ansible/hosts` 下。一些示例被提供并注释掉了：

```text
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers:

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group:

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern, you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group:

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

## db-[99:101]-node.example.com
```

如你所见，作为示例提供的文件使用 INI 格式，这是系统管理员熟知的格式。请注意，你可以选择其他文件格式（例如 yaml），但对于初步测试，INI 格式很适合我们后续的示例。

清单可以在生产中自动生成，特别是如果你有像 VMware VSphere 这样的虚拟化环境或云环境（Aws、OpenStack 或其他）。

* 在 `/etc/ansible/hosts` 中创建主机组：

正如你可能已经注意到的，组在方括号中声明。然后是属于这些组的元素。你可以通过在此文件中插入以下块来创建例如 `rocky8` 组：

```bash
[rocky8]
172.16.1.10
172.16.1.11
```

组可以在其他组中使用。在这种情况下，必须使用 `:children` 属性指定父组由子组组成，如下所示：

```bash
[linux:children]
rocky8
debian9

[ansible:children]
ansible_management
ansible_clients

[ansible_management]
172.16.1.10

[ansible_clients]
172.16.1.10
```

我们不会进一步深入清单，但如果你感兴趣，请考虑查看[此链接](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)。

现在我们的管理服务器已安装，清单已准备就绪，是时候运行我们的第一个 `ansible` 命令了。

## `ansible` 命令行使用

`ansible` 命令在一个或多个目标主机上启动一个任务。

```bash
ansible <host-pattern> [-m module_name] [-a args] [options]
```

示例：

!!! Warning

    由于我们尚未在我们的 2 台测试服务器上配置认证，并非所有以下示例都能正常工作。它们作为示例来帮助理解，并将在本章稍后完全可用。

* 列出属于 rocky8 组的主机：

```bash
ansible rocky8 --list-hosts
```

* 使用 `ping` 模块 ping 一个主机组：

```bash
ansible rocky8 -m ping
```

* 使用 `setup` 模块显示主机组的事实（facts）：

```bash
ansible rocky8 -m setup
```

* 通过调用 `command` 模块并带参数在主机组上运行命令：

```bash
ansible rocky8 -m command -a 'uptime'
```

* 以管理员权限运行命令：

```bash
ansible ansible_clients --become -m command -a 'reboot'
```

* 使用自定义清单文件运行命令：

```bash
ansible rocky8 -i ./local-inventory -m command -a 'date'
```

!!! Note

    如本例所示，有时将受管设备的声明分成多个文件（例如按云项目）并为 Ansible 提供这些文件的路径，比维护一个长的清单文件更简单。

| 选项                      | 信息                                                                         |
|--------------------------|-----------------------------------------------------------------------------|
| `-a 'arguments'`         | 传递给模块的参数。                                                             |
| `-b -K`                  | 请求密码并以更高权限运行命令。                                                    |
| `--user=username`        | 使用此用户连接到目标主机，而不是当前用户。                                           |
| `--become-user=username` | 以此用户身份执行操作（默认：`root`）。                                             |
| `-C`                     | 模拟。不对目标进行任何更改，但测试应该更改的内容。                                     |
| `-m module`              | 运行指定的模块                                                                  |

### 准备客户端

在管理机和客户端上，我们将创建一个 `ansible` 用户，专门用于 Ansible 执行的操作。此用户将需要使用 sudo 权限，因此必须将其添加到 `wheel` 组。

此用户将用于：

* 在管理站端：运行 `ansible` 命令并通过 SSH 连接到被管理客户端。
* 在被管理站端（这里作为管理站的服务器也充当客户端，因此它由自身管理）执行从管理站启动的命令：因此它必须具有 sudo 权限。

在两台机器上，创建一个专用于 ansible 的 `ansible` 用户：

```bash
sudo useradd ansible
sudo usermod -aG wheel ansible
```

为此用户设置密码：

```bash
sudo passwd ansible
```

修改 sudoers 配置以允许 `wheel` 组的成员无需密码即可 sudo：

```bash
sudo visudo
```

我们的目标是注释掉默认行，并取消注释 NOPASSWD 选项，使得完成后这些行如下所示：

```bash
## Allows people in group wheel to run all commands
# %wheel  ALL=(ALL)       ALL

## Same thing without a password
%wheel        ALL=(ALL)       NOPASSWD: ALL
```

!!! Warning

    如果在输入 Ansible 命令时收到以下错误消息，则可能意味着你在某台客户端上忘记了此步骤：
    `"msg": "Missing sudo password`

从此时起使用管理功能时，开始使用此新用户：

```bash
sudo su - ansible
```

### 使用 ping 模块测试

默认情况下，Ansible 不允许密码登录。

取消注释 `/etc/ansible/ansible.cfg` 配置文件中 `[defaults]` 部分中的以下行并将其设置为 True：

```bash
ask_pass      = True
```

对 rocky8 组的每台服务器运行 `ping`：

```bash
# ansible rocky8 -m ping
SSH password:
172.16.1.10 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
172.16.1.11 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

!!! Note

    系统会要求你输入远程服务器的 `ansible` 密码，这是一个安全问题……

!!! Tip

    如果你收到此错误 `"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"`，你可以在管理站上安装 `sshpass`：

    ```
    $ sudo dnf install sshpass
    ```

!!! Abstract

    你现在可以测试本章之前无法工作的命令了。

## 密钥认证

密码认证将被更安全的私钥/公钥认证所取代。

### 创建 SSH 密钥

双密钥将由 `ansible` 用户在管理站上使用 `ssh-keygen` 命令生成：

```bash
[ansible]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ansible/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ansible/.ssh/id_rsa.
Your public key has been saved in /home/ansible/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Oa1d2hYzzdO0e/K10XPad25TA1nrSVRPIuS4fnmKr9g ansible@localhost.localdomain
The key's randomart image is:
+---[RSA 3072]----+
|           .o . +|
|           o . =.|
|          . . + +|
|         o . = =.|
|        S o = B.o|
|         = + = =+|
|        . + = o+B|
|         o + o *@|
|        . Eoo .+B|
+----[SHA256]-----+

```

公钥可以复制到服务器上：

```bash
# ssh-copy-id ansible@172.16.1.10
# ssh-copy-id ansible@172.16.1.11
```

重新注释 `/etc/ansible/ansible.cfg` 配置文件中 `[defaults]` 部分的以下行，以防止密码认证：

```bash
#ask_pass      = True
```

### 私钥认证测试

对于下一个测试，使用 `shell` 模块，它允许远程执行命令：

```bash
# ansible rocky8 -m shell -a "uptime"
172.16.1.10 | SUCCESS | rc=0 >>
 12:36:18 up 57 min,  1 user,  load average: 0.00, 0.00, 0.00

172.16.1.11 | SUCCESS | rc=0 >>
 12:37:07 up 57 min,  1 user,  load average: 0.00, 0.00, 0.00
```

无需密码，私钥/公钥认证工作正常！

!!! Note

    在生产环境中，你现在应该移除之前设置的 `ansible` 密码以加强你的安全性（因为现在不需要认证密码了）。

## 使用 Ansible

Ansible 可以从 shell 使用或通过 playbook 使用。

### 模块

按类别分类的模块列表可以在[此处找到](https://docs.ansible.com/ansible/latest/collections/index_module.html)。Ansible 提供了超过 750 个！

模块现在被分组为模块集合，其列表可以在[此处找到](https://docs.ansible.com/ansible/latest/collections/index.html)。

集合是 Ansible 内容的分发格式，可以包括 playbook、角色、模块和插件。

使用 `ansible` 命令的 `-m` 选项调用模块：

```bash
ansible <host-pattern> [-m module_name] [-a args] [options]
```

几乎每个需求都有对应的模块！因此，建议使用适合需求的模块，而不是使用 shell 模块。

每个需求类别都有自己的模块。以下是一个非详尽的列表：

| 类型                 | 示例                                                          |
|---------------------|---------------------------------------------------------------|
| 系统管理              | `user`（用户管理）、`group`（组管理）等。                           |
| 软件管理              | `dnf`、`yum`、`apt`、`pip`、`npm`                              |
| 文件管理              | `copy`、`fetch`、`lineinfile`、`template`、`archive`           |
| 数据库管理            | `mysql`、`postgresql`、`redis`                                   |
| 云管理                | `amazon S3`、`cloudstack`、`openstack`                          |
| 集群管理              | `consul`、`zookeeper`                                          |
| 发送命令              | `shell`、`script`、`expect`                                    |
| 下载                  | `get_url`                                                      |
| 源代码管理            | `git`、`gitlab`                                                |

#### 软件安装示例

`dnf` 模块允许在目标客户端上安装软件：

```bash
# ansible rocky8 --become -m dnf -a name="httpd"
172.16.1.10 | SUCCESS => {
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
      ...
      \n\nComplete!\n"
    ]
}
172.16.1.11 | SUCCESS => {
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
      ...
    \n\nComplete!\n"
    ]
}
```

安装的软件是一个服务，现在需要使用 `systemd` 模块启动它：

```bash
# ansible rocky8 --become  -m systemd -a "name=httpd state=started"
172.16.1.10 | SUCCESS => {
    "changed": true,
    "name": "httpd",
    "state": "started"
}
172.16.1.11 | SUCCESS => {
    "changed": true,
    "name": "httpd",
    "state": "started"
}
```

!!! Tip

    尝试两次运行最后 2 个命令。你会发现第一次 Ansible 将采取行动以达到命令设置的状态。第二次，它将什么也不做，因为它检测到状态已经达到了！

### 练习

为了帮助发现更多关于 Ansible 的内容，并习惯于搜索 Ansible 文档，这里有一些你在继续之前可以做的练习：

* 创建组 Paris、Tokio、NewYork
* 创建用户 `supervisor`
* 更改此用户的 uid 为 10000
* 更改用户使其属于 Paris 组
* 安装 tree 软件
* 停止 crond 服务
* 创建一个权限为 `644` 的空文件
* 更新你的客户端发行版
* 重启你的客户端

!!! Warning

    不要使用 shell 模块。在文档中查找适当的模块！

#### `setup` 模块：事实介绍

系统事实是由 Ansible 通过其 `setup` 模块检索的变量。

查看客户端的不同事实，了解通过一个简单的命令可以轻松检索多少信息。

我们稍后将看到如何在 playbook 中使用事实以及如何创建我们自己的事实。

```bash
# ansible ansible_clients -m setup | less
192.168.1.11 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.1.11"
        ],
        "ansible_all_ipv6_addresses": [
            "2001:861:3dc3:fcf0:a00:27ff:fef7:28be",
            "fe80::a00:27ff:fef7:28be"
        ],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "12/01/2006",
        "ansible_bios_vendor": "innotek GmbH",
        "ansible_bios_version": "VirtualBox",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "VirtualBox",
        "ansible_board_serial": "NA",
        "ansible_board_vendor": "Oracle Corporation",
        ...
```

现在我们已经了解了如何在命令行上使用 Ansible 配置远程服务器，我们可以介绍 playbook 的概念。Playbook 是使用 Ansible 的另一种方式，并不复杂得多，但会使代码重用更加容易。

## Playbook（剧本）

Ansible 的 playbook 描述了应用于远程系统的策略，以强制执行其配置。Playbook 使用易于理解的文本格式编写，将一组任务组合在一起：`yaml` 格式。

!!! Note

    在[此处](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)了解更多关于 yaml 的信息。

```bash
ansible-playbook <file.yml> ... [options]
```

选项与 `ansible` 命令相同。

该命令返回以下错误代码：

| 代码   | 错误                             |
|-------|----------------------------------|
| `0`   | OK 或没有匹配的主机                |
| `1`   | 错误                             |
| `2`   | 一个或多个主机失败                  |
| `3`   | 一个或多个主机不可达                |
| `4`   | 分析错误                           |
| `5`   | 错误或不完整的选项                  |
| `99`  | 运行被用户中断                      |
| `250` | 意外错误                           |

!!! Note

    请注意，当没有主机匹配你的目标时，`ansible` 将返回 Ok，这可能会误导你！

### Apache 和 MySQL playbook 示例

以下 playbook 允许我们在目标服务器上安装 Apache 和 MariaDB。

创建一个包含以下内容的 `test.yml` 文件：

```bash
---
- hosts: rocky8 <1>
  become: true <2>
  become_user: root

  tasks:

    - name: ensure apache is at the latest version
      dnf: name=httpd,php,php-mysqli state=latest

    - name: ensure httpd is started
      systemd: name=httpd state=started

    - name: ensure mariadb is at the latest version
      dnf: name=mariadb-server state=latest

    - name: ensure mariadb is started
      systemd: name=mariadb state=started
...
```

* <1> 目标组或目标服务器必须存在于清单中
* <2> 连接后，用户成为 `root`（默认通过 `sudo`）

使用 `ansible-playbook` 命令执行 playbook：

```bash
$ ansible-playbook test.yml

PLAY [rocky8] ****************************************************************

TASK [setup] ******************************************************************
ok: [172.16.1.10]
ok: [172.16.1.11]

TASK [ensure apache is at the latest version] *********************************
ok: [172.16.1.10]
ok: [172.16.1.11]

TASK [ensure httpd is started] ************************************************
changed: [172.16.1.10]
changed: [172.16.1.11]

TASK [ensure mariadb is at the latest version] **********************************
changed: [172.16.1.10]
changed: [172.16.1.11]

TASK [ensure mariadb is started] ***********************************************
changed: [172.16.1.10]
changed: [172.16.1.11]

PLAY RECAP *********************************************************************
172.16.1.10             : ok=5    changed=3    unreachable=0    failed=0
172.16.1.11             : ok=5    changed=3    unreachable=0    failed=0
```

为了提高可读性，建议以完整的 yaml 格式编写你的 playbook。在前面的示例中，参数与模块在同一行给出，参数值通过 `=` 与其名称分隔。查看完整 yaml 格式的相同 playbook：

```bash
---
- hosts: rocky8
  become: true
  become_user: root

  tasks:

    - name: ensure apache is at the latest version
      dnf:
        name: httpd,php,php-mysqli
        state: latest

    - name: ensure httpd is started
      systemd:
        name: httpd
        state: started

    - name: ensure mariadb is at the latest version
      dnf:
        name: mariadb-server
        state: latest

    - name: ensure mariadb is started
      systemd:
        name: mariadb
        state: started
...
```

!!! Tip

    `dnf` 是允许你给它一个列表作为参数的模块之一。

关于集合的说明：Ansible 现在以集合的形式提供模块。
一些模块默认在 `ansible.builtin` 集合中提供，其他模块必须通过以下命令手动安装：

```bash
ansible-galaxy collection install [collectionname]
```

其中 [collectionname] 是集合的名称（这里的方括号用于强调需要将其替换为实际的集合名称，而不是命令的一部分）。

前面的示例应该像这样编写：

```bash
---
- hosts: rocky8
  become: true
  become_user: root

  tasks:

    - name: ensure apache is at the latest version
      ansible.builtin.dnf:
        name: httpd,php,php-mysqli
        state: latest

    - name: ensure httpd is started
      ansible.builtin.systemd:
        name: httpd
        state: started

    - name: ensure mariadb is at the latest version
      ansible.builtin.dnf:
        name: mariadb-server
        state: latest

    - name: ensure mariadb is started
      ansible.builtin.systemd:
        name: mariadb
        state: started
...
```

一个 playbook 不限于一个目标：

```bash
---
- hosts: webservers
  become: true
  become_user: root

  tasks:

    - name: ensure apache is at the latest version
      ansible.builtin.dnf:
        name: httpd,php,php-mysqli
        state: latest

    - name: ensure httpd is started
      ansible.builtin.systemd:
        name: httpd
        state: started

- hosts: databases
  become: true
  become_user: root

    - name: ensure mariadb is at the latest version
      ansible.builtin.dnf:
        name: mariadb-server
        state: latest

    - name: ensure mariadb is started
      ansible.builtin.systemd:
        name: mariadb
        state: started
...
```

你可以检查 playbook 的语法：

```bash
ansible-playbook --syntax-check play.yml
```

你也可以使用 yaml 的 "linter"：

```bash
dnf install -y yamllint
```

然后检查 playbook 的 yaml 语法：

```bash
$ yamllint test.yml
test.yml
  8:1       error    syntax error: could not find expected ':' (syntax)
```

## 练习结果

* 创建组 Paris、Tokio、NewYork
* 创建用户 `supervisor`
* 更改此用户的 uid 为 10000
* 更改用户使其属于 Paris 组
* 安装 tree 软件
* 停止 crond 服务
* 创建一个权限为 `0644` 的空文件
* 更新你的客户端发行版
* 重启你的客户端

```bash
ansible ansible_clients --become -m group -a "name=Paris"
ansible ansible_clients --become -m group -a "name=Tokio"
ansible ansible_clients --become -m group -a "name=NewYork"
ansible ansible_clients --become -m user -a "name=Supervisor"
ansible ansible_clients --become -m user -a "name=Supervisor uid=10000"
ansible ansible_clients --become -m user -a "name=Supervisor uid=10000 groups=Paris"
ansible ansible_clients --become -m dnf -a "name=tree"
ansible ansible_clients --become -m systemd -a "name=crond state=stopped"
ansible ansible_clients --become -m copy -a "content='' dest=/tmp/test force=no mode=0644"
ansible ansible_clients --become -m dnf -a "name=* state=latest"
ansible ansible_clients --become -m reboot
```
