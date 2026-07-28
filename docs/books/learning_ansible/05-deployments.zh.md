---
title: 使用 Ansistrano 部署
---

# 使用 Ansistrano 进行 Ansible 部署

在本章中，你将学习如何使用 Ansible 角色 [Ansistrano](https://ansistrano.com) 部署应用程序。

****

**目标**：在本章中你将学习：

:heavy_check_mark: 实现 Ansistrano；
:heavy_check_mark: 配置 Ansistrano；
:heavy_check_mark: 在部署版本之间使用共享文件夹和文件；
:heavy_check_mark: 从 git 部署站点的不同版本；
:heavy_check_mark: 在部署步骤之间做出反应。

:checkered_flag: **ansible**、**ansistrano**、**roles**、**deployments**

**知识**：:star: :star:
**复杂度**：:star: :star: :star:

**阅读时间**：40 分钟

****

Ansistrano 是一个 Ansible 角色，用于轻松部署 PHP、Python 等应用程序。
它基于 [Capistrano](http://capistranorb.com/) 的功能。

## 简介

Ansistrano 运行需要以下条件：

* 部署机器上的 Ansible，
* 客户端机器上的 `rsync` 或 `git`。

它可以从 `rsync`、`git`、`scp`、`http`、`S3` 等下载源代码。

!!! Note

    对于我们的部署示例，我们将使用 `git` 协议。

Ansistrano 通过以下 5 个步骤部署应用程序：

* **Setup**：创建用于存放发布（releases）的目录结构；
* **Update Code**：将新版本下载到目标；
* **Symlink Shared** 和 **Symlink**：部署新版本后，`current` 符号链接被修改以指向此新版本；
* **Clean Up**：进行一些清理工作（删除旧版本）。

![部署的阶段](images/ansistrano-001.png)

使用 Ansistrano 部署的骨架如下所示：

```bash
/var/www/site/
├── current -> ./releases/20210718100000Z
├── releases
│   └── 20210718100000Z
│       ├── css -> ../../shared/css/
│       ├── img -> ../../shared/img/
│       └── REVISION
├── repo
└── shared
    ├── css/
    └── img/
```

你可以在其 [Github 仓库](https://github.com/ansistrano/deploy)中找到所有 Ansistrano 文档。

## 实验

你将在你的 2 台服务器上继续工作：

管理服务器：

* Ansible 已经安装。你需要安装 `ansistrano.deploy` 角色。

被管理服务器：

* 你需要安装 Apache 并部署客户端站点。

### 部署 Web 服务器

为了提高效率，我们将使用 `geerlingguy.apache` 角色来配置服务器：

```bash
$ ansible-galaxy role install geerlingguy.apache
Starting galaxy role install process
- downloading role 'apache', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-apache/archive/3.1.4.tar.gz
- extracting geerlingguy.apache to /home/ansible/.ansible/roles/geerlingguy.apache
- geerlingguy.apache (3.1.4) was installed successfully
```

我们可能需要打开一些防火墙规则，因此我们还将安装集合 `ansible.posix` 以使用其模块 `firewalld`：

```bash
$ ansible-galaxy collection install ansible.posix
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/ansible-posix-1.2.0.tar.gz to /home/ansible/.ansible/tmp/ansible-local-519039bp65pwn/tmpsvuj1fw5/ansible-posix-1.2.0-bhjbfdpw
Installing 'ansible.posix:1.2.0' to '/home/ansible/.ansible/collections/ansible_collections/ansible/posix'
ansible.posix:1.2.0 was installed successfully
```

一旦角色和集合安装完成，我们可以创建 playbook 的第一部分，它将：

* 安装 Apache，
* 为我们的 `vhost`（虚拟主机）创建目标文件夹，
* 创建一个默认的 `vhost`，
* 打开防火墙，
* 启动或重启 Apache。

技术考虑：

* 我们将把我们的站点部署到 `/var/www/site/` 文件夹。
* 正如我们稍后将看到的，`ansistrano` 将创建一个 `current` 符号链接指向当前发布文件夹。
* 待部署的源代码包含一个 `html` 文件夹，vhost 应指向该文件夹。其 `DirectoryIndex` 是 `index.htm`。
* 部署通过 `git` 完成，该软件包将被安装。

!!! Note

    因此，我们的 vhost 的目标将是：`/var/www/site/current/html`。

我们的配置服务器的 playbook：`playbook-config-server.yml`

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    apache_global_vhost_settings: |
      DirectoryIndex index.php index.htm
    apache_vhosts:
      - servername: "website"
    documentroot: "{{ dest }}current/html"

  tasks:

    - name: create directory for website
      file:
        path: /var/www/site/
        state: directory
        mode: 0755

    - name: install git
      package:
        name: git
        state: latest

    - name: permit traffic in default zone for http service
      ansible.posix.firewalld:
        service: http
        permanent: yes
        state: enabled
        immediate: yes

  roles:
    - { role: geerlingguy.apache }
```

playbook 可以应用到服务器：

```bash
ansible-playbook playbook-config-server.yml
```

注意以下任务的执行：

```bash
TASK [geerlingguy.apache : Ensure Apache is installed on RHEL.] ****************
TASK [geerlingguy.apache : Configure Apache.] **********************************
TASK [geerlingguy.apache : Add apache vhosts configuration.] *******************
TASK [geerlingguy.apache : Ensure Apache has selected state and enabled on boot.] ***
TASK [permit traffic in default zone for http service] *************************
RUNNING HANDLER [geerlingguy.apache : restart apache] **************************
```

`geerlingguy.apache` 角色通过负责 Apache 的安装和配置使我们的工作变得更加容易。

你可以使用 `curl` 检查一切是否正常工作：

```bash
$ curl -I http://192.168.1.11
HTTP/1.1 404 Not Found
Date: Mon, 05 Jul 2021 23:30:02 GMT
Server: Apache/2.4.37 (rocky) OpenSSL/1.1.1g
Content-Type: text/html; charset=iso-8859-1
```

!!! Note

    我们还没有部署任何代码，所以 `curl` 返回 `404` HTTP 代码是正常的。但我们已经可以确认 `httpd` 服务正在运行，并且防火墙是打开的。

### 部署软件

现在我们的服务器已配置好，我们可以部署应用程序。

为此，我们将在专门用于应用程序部署的第二个 playbook 中使用 `ansistrano.deploy` 角色（为了提高可读性）。

```bash
$ ansible-galaxy role install ansistrano.deploy
Starting galaxy role install process
- downloading role 'deploy', owned by ansistrano
- downloading role from https://github.com/ansistrano/deploy/archive/3.10.0.tar.gz
- extracting ansistrano.deploy to /home/ansible/.ansible/roles/ansistrano.deploy
- ansistrano.deploy (3.10.0) was installed successfully

```

软件的源代码可以在 [github 仓库](https://github.com/alemorvan/demo-ansible.git)中找到。

我们将创建一个 playbook `playbook-deploy.yml` 来管理我们的部署：

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    ansistrano_deploy_via: "git"
    ansistrano_git_repo: https://github.com/alemorvan/demo-ansible.git
    ansistrano_deploy_to: "{{ dest }}"

  roles:
     - { role: ansistrano.deploy }
```

```bash
$ ansible-playbook playbook-deploy.yml

PLAY [ansible_clients] *********************************************************

TASK [ansistrano.deploy : ANSISTRANO | Ensure deployment base path exists] *****
TASK [ansistrano.deploy : ANSISTRANO | Ensure releases folder exists]
TASK [ansistrano.deploy : ANSISTRANO | Ensure shared elements folder exists]
TASK [ansistrano.deploy : ANSISTRANO | Ensure shared paths exists]
TASK [ansistrano.deploy : ANSISTRANO | Ensure basedir shared files exists]
TASK [ansistrano.deploy : ANSISTRANO | Get release version] ********************
TASK [ansistrano.deploy : ANSISTRANO | Get release path]
TASK [ansistrano.deploy : ANSISTRANO | GIT | Register ansistrano_git_result variable]
TASK [ansistrano.deploy : ANSISTRANO | GIT | Set git_real_repo_tree]
TASK [ansistrano.deploy : ANSISTRANO | GIT | Create release folder]
TASK [ansistrano.deploy : ANSISTRANO | GIT | Sync repo subtree[""] to release path]
TASK [ansistrano.deploy : ANSISTRANO | Copy git released version into REVISION file]
TASK [ansistrano.deploy : ANSISTRANO | Ensure shared paths targets are absent]
TASK [ansistrano.deploy : ANSISTRANO | Create softlinks for shared paths and files]
TASK [ansistrano.deploy : ANSISTRANO | Ensure .rsync-filter is absent]
TASK [ansistrano.deploy : ANSISTRANO | Setup .rsync-filter with shared-folders]
TASK [ansistrano.deploy : ANSISTRANO | Get current folder]
TASK [ansistrano.deploy : ANSISTRANO | Remove current folder if it's a directory]
TASK [ansistrano.deploy : ANSISTRANO | Change softlink to new release]
TASK [ansistrano.deploy : ANSISTRANO | Clean up releases]

PLAY RECAP ********************************************************************************************************************************************************************************************************
192.168.1.11 : ok=25   changed=8    unreachable=0    failed=0    skipped=14   rescued=0    ignored=0

```

仅用 11 行代码就完成了这么多事情！

```html
$ curl http://192.168.1.11
<html>
<head>
<title>Demo Ansible</title>
</head>
<body>
<h1>Version Master</h1>
</body>
<html>
```

### 检查服务器上的内容

你现在可以通过 ssh 连接到你的客户端机器。

* 对 `/var/www/site/` 目录运行 `tree`：

```bash
$ tree /var/www/site/
/var/www/site
├── current -> ./releases/20210722155312Z
├── releases
│   └── 20210722155312Z
│       ├── REVISION
│       └── html
│       └── index.htm
├── repo
│   └── html
│       └── index.htm
└── shared
```

请注意：

* `current` 符号链接指向发布 `./releases/20210722155312Z`
* 存在一个 `shared` 目录
* 在 `./repo/` 中存在 git 仓库

* 从 Ansible 服务器，重新启动部署 **3** 次，然后在客户端检查。

```bash
$ tree /var/www/site/
var/www/site
├── current -> ./releases/20210722160048Z
├── releases
│   ├── 20210722155312Z
│   │   ├── REVISION
│   │   └── html
│   │       └── index.htm
│   ├── 20210722160032Z
│   │   ├── REVISION
│   │   └── html
│   │       └── index.htm
│   ├── 20210722160040Z
│   │   ├── REVISION
│   │   └── html
│   │       └── index.htm
│   └── 20210722160048Z
│       ├── REVISION
│       └── html
│       └── index.htm
├── repo
│   └── html
│       └── index.htm
└── shared
```

请注意：

* `ansistrano` 保留了最近 4 个发布，
* `current` 链接现在链接到最新的发布

### 限制发布数量

`ansistrano_keep_releases` 变量用于指定要保持的发布数量。

* 使用 `ansistrano_keep_releases` 变量，仅保留 3 个项目的发布。检查。

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    ansistrano_deploy_via: "git"
    ansistrano_git_repo: https://github.com/alemorvan/demo-ansible.git
    ansistrano_deploy_to: "{{ dest }}"
    ansistrano_keep_releases: 3

  roles:
     - { role: ansistrano.deploy }
```

```bash
---
$ ansible-playbook -i hosts playbook-deploy.yml
```

在客户端机器上：

```bash
$ tree /var/www/site/
/var/www/site
├── current -> ./releases/20210722160318Z
├── releases
│   ├── 20210722160040Z
│   │   ├── REVISION
│   │   └── html
│   │       └── index.htm
│   ├── 20210722160048Z
│   │   ├── REVISION
│   │   └── html
│   │       └── index.htm
│   └── 20210722160318Z
│       ├── REVISION
│       └── html
│       └── index.htm
├── repo
│   └── html
│       └── index.htm
└── shared
```

### 使用 shared_paths 和 shared_files

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    ansistrano_deploy_via: "git"
    ansistrano_git_repo: https://github.com/alemorvan/demo-ansible.git
    ansistrano_deploy_to: "{{ dest }}"
    ansistrano_keep_releases: 3
    ansistrano_shared_paths:
      - "img"
      - "css"
    ansistrano_shared_files:
      - "logs"

  roles:
     - { role: ansistrano.deploy }
```

在客户端机器上，在 `shared` 目录中创建文件 `logs`：

```bash
sudo touch /var/www/site/shared/logs
```

然后执行 playbook：

```bash
TASK [ansistrano.deploy : ANSISTRANO | Ensure shared paths targets are absent] *******************************************************
ok: [192.168.10.11] => (item=img)
ok: [192.168.10.11] => (item=css)
ok: [192.168.10.11] => (item=logs/log)

TASK [ansistrano.deploy : ANSISTRANO | Create softlinks for shared paths and files] **************************************************
changed: [192.168.10.11] => (item=img)
changed: [192.168.10.11] => (item=css)
changed: [192.168.10.11] => (item=logs)
```

在客户端机器上：

```bash
$  tree -F /var/www/site/
/var/www/site/
├── current -> ./releases/20210722160631Z/
├── releases/
│   ├── 20210722160048Z/
│   │   ├── REVISION
│   │   └── html/
│   │       └── index.htm
│   ├── 20210722160318Z/
│   │   ├── REVISION
│   │   └── html/
│   │       └── index.htm
│   └── 20210722160631Z/
│       ├── REVISION
│       ├── css -> ../../shared/css/
│       ├── html/
│       │   └── index.htm
│       ├── img -> ../../shared/img/
│       └── logs -> ../../shared/logs
├── repo/
│   └── html/
│       └── index.htm
└── shared/
    ├── css/
    ├── img/
    └── logs
```

请注意，最新版本包含 3 个链接：`css`、`img` 和 `logs`

* 从 `/var/www/site/releases/css` 到 `../../shared/css/` 目录。
* 从 `/var/www/site/releases/img` 到 `../../shared/img/` 目录。
* 从 `/var/www/site/releases/logs` 到 `../../shared/logs` 文件。

因此，这 2 个文件夹中包含的文件和 `logs` 文件始终可以通过以下路径访问：

* `/var/www/site/current/css/`，
* `/var/www/site/current/img/`，
* `/var/www/site/current/logs`，

但最重要的是，它们将从一个发布保留到下一个发布。

### 使用仓库的子目录进行部署

在我们的例子中，仓库包含一个 `html` 文件夹，其中包含站点文件。

* 为了避免这个额外的目录层级，使用 `ansistrano_git_repo_tree` 变量，指定要使用的子目录的路径。

不要忘记修改 Apache 配置以考虑此更改！

更改服务器配置的 playbook `playbook-config-server.yml`

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    apache_global_vhost_settings: |
      DirectoryIndex index.php index.htm
    apache_vhosts:
      - servername: "website"
    documentroot: "{{ dest }}current/" # <1>

  tasks:

    - name: create directory for website
      file:
        path: /var/www/site/
        state: directory
        mode: 0755

    - name: install git
      package:
        name: git
        state: latest

  roles:
    - { role: geerlingguy.apache }
```

<1> 修改此行

更改部署的 playbook `playbook-deploy.yml`

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    ansistrano_deploy_via: "git"
    ansistrano_git_repo: https://github.com/alemorvan/demo-ansible.git
    ansistrano_deploy_to: "{{ dest }}"
    ansistrano_keep_releases: 3
    ansistrano_shared_paths:
      - "img"
      - "css"
    ansistrano_shared_files:
      - "log"
    ansistrano_git_repo_tree: 'html' # <1>

  roles:
     - { role: ansistrano.deploy }
```

<1> 修改此行

* 不要忘记运行两个 playbook

* 在客户端机器上检查：

```bash
$  tree -F /var/www/site/
/var/www/site/
├── current -> ./releases/20210722161542Z/
├── releases/
│   ├── 20210722160318Z/
│   │   ├── REVISION
│   │   └── html/
│   │       └── index.htm
│   ├── 20210722160631Z/
│   │   ├── REVISION
│   │   ├── css -> ../../shared/css/
│   │   ├── html/
│   │   │   └── index.htm
│   │   ├── img -> ../../shared/img/
│   │   └── logs -> ../../shared/logs
│   └── 20210722161542Z/
│       ├── REVISION
│       ├── css -> ../../shared/css/
│       ├── img -> ../../shared/img/
│       ├── index.htm
│       └── logs -> ../../shared/logs
├── repo/
│   └── html/
│       └── index.htm
└── shared/
    ├── css/
    ├── img/
    └── logs
```

<1> 请注意没有 `html` 目录

### 管理 git 分支或标签

`ansistrano_git_branch` 变量用于指定要部署的 `branch`（分支）或 `tag`（标签）。

* 部署 `releases/v1.1.0` 分支：

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    ansistrano_deploy_via: "git"
    ansistrano_git_repo: https://github.com/alemorvan/demo-ansible.git
    ansistrano_deploy_to: "{{ dest }}"
    ansistrano_keep_releases: 3
    ansistrano_shared_paths:
      - "img"
      - "css"
    ansistrano_shared_files:
      - "log"
    ansistrano_git_repo_tree: 'html'
    ansistrano_git_branch: 'releases/v1.1.0'

  roles:
     - { role: ansistrano.deploy }
```

!!! Note

    在部署期间，你可以刷新你的浏览器来"实时"查看更改。

```html
$ curl http://192.168.1.11
<html>
<head>
<title>Demo Ansible</title>
</head>
<body>
<h1>Version 1.0.1</h1>
</body>
<html>
```

* 部署 `v2.0.0` 标签：

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    ansistrano_deploy_via: "git"
    ansistrano_git_repo: https://github.com/alemorvan/demo-ansible.git
    ansistrano_deploy_to: "{{ dest }}"
    ansistrano_keep_releases: 3
    ansistrano_shared_paths:
      - "img"
      - "css"
    ansistrano_shared_files:
      - "log"
    ansistrano_git_repo_tree: 'html'
    ansistrano_git_branch: 'v2.0.0'

  roles:
     - { role: ansistrano.deploy }
```

```html
$ curl http://192.168.1.11
<html>
<head>
<title>Demo Ansible</title>
</head>
<body>
<h1>Version 2.0.0</h1>
</body>
<html>
```

### 部署步骤之间的动作

使用 Ansistrano 进行部署遵循以下步骤：

* `Setup`
* `Update Code`
* `Symlink Shared`
* `Symlink`
* `Clean Up`

可以在每个步骤之前和之后介入。

![部署的阶段](images/ansistrano-001.png)

可以通过为此目的提供的变量来包含 playbook：

* `ansistrano_before_<task>_tasks_file`
* 或 `ansistrano_after_<task>_tasks_file`

* 简单示例：在部署开始时发送一封电子邮件（或任何你想要的内容，如 Slack 通知）：

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    ansistrano_deploy_via: "git"
    ansistrano_git_repo: https://github.com/alemorvan/demo-ansible.git
    ansistrano_deploy_to: "{{ dest }}"
    ansistrano_keep_releases: 3
    ansistrano_shared_paths:
      - "img"
      - "css"
    ansistrano_shared_files:
      - "logs"
    ansistrano_git_repo_tree: 'html'
    ansistrano_git_branch: 'v2.0.0'
    ansistrano_before_setup_tasks_file: "{{ playbook_dir }}/deploy/before-setup-tasks.yml"

  roles:
     - { role: ansistrano.deploy }
```

创建文件 `deploy/before-setup-tasks.yml`：

```yaml
---
- name: Send a mail
  mail:
    subject: Starting deployment on {{ ansible_hostname }}.
  delegate_to: localhost
```

```bash
TASK [ansistrano.deploy : include] *************************************************************************************
included: /home/ansible/deploy/before-setup-tasks.yml for 192.168.10.11

TASK [ansistrano.deploy : Send a mail] *************************************************************************************
ok: [192.168.10.11 -> localhost]
```

```bash
[root] # mailx
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 1 message 1 new
>N  1 root@localhost.local  Tue Aug 21 14:41  28/946   "Starting deployment on localhost."
```

* 你可能需要在部署结束时重新启动一些服务，例如刷新缓存。让我们在部署结束时重新启动 Apache：

```yaml
---
- hosts: ansible_clients
  become: yes
  become_user: root
  vars:
    dest: "/var/www/site/"
    ansistrano_deploy_via: "git"
    ansistrano_git_repo: https://github.com/alemorvan/demo-ansible.git
    ansistrano_deploy_to: "{{ dest }}"
    ansistrano_keep_releases: 3
    ansistrano_shared_paths:
      - "img"
      - "css"
    ansistrano_shared_files:
      - "logs"
    ansistrano_git_repo_tree: 'html'
    ansistrano_git_branch: 'v2.0.0'
    ansistrano_before_setup_tasks_file: "{{ playbook_dir }}/deploy/before-setup-tasks.yml"
    ansistrano_after_symlink_tasks_file: "{{ playbook_dir }}/deploy/after-symlink-tasks.yml"

  roles:
     - { role: ansistrano.deploy }
```

创建文件 `deploy/after-symlink-tasks.yml`：

```yaml
---
- name: restart apache
  systemd:
    name: httpd
    state: restarted
```

```bash
TASK [ansistrano.deploy : include] *************************************************************************************
included: /home/ansible/deploy/after-symlink-tasks.yml for 192.168.10.11

TASK [ansistrano.deploy : restart apache] **************************************************************************************
changed: [192.168.10.11]
```

正如你在本章中看到的，Ansible 可以极大地改善系统管理员的生活。像 Ansistrano 这样非常智能的角色是"必备品"，很快就会变得不可或缺。

使用 Ansistrano 确保良好的部署实践得到尊重，减少将系统投入生产所需的时间，并避免潜在的人为错误风险。机器工作速度快、质量好，而且很少犯错！
