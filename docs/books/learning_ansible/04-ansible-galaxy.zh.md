---
title: Ansible Galaxy
---

# Ansible Galaxy：集合与角色

在本章中，你将学习如何使用、安装和管理 Ansible 角色（roles）和集合（collections）。

****

**目标**：在本章中你将学习：

:heavy_check_mark: 安装和管理集合。
:heavy_check_mark: 安装和管理角色。

:checkered_flag: **ansible**、**ansible-galaxy**、**roles**、**collections**

**知识**：:star: :star:
**复杂度**：:star: :star: :star:

**阅读时间**：40 分钟

****

[Ansible Galaxy](https://galaxy.ansible.com) 提供来自 Ansible Community 的 Ansible 角色和集合。

提供的元素可以在 playbook 中引用并立即可用。

## `ansible-galaxy` 命令

`ansible-galaxy` 命令使用 [galaxy.ansible.com](http://galaxy.ansible.com) 管理角色和集合。

* 管理角色：

```bash
ansible-galaxy role [import|init|install|login|remove|...]
```

| 子命令        | 功能                                   |
|--------------|----------------------------------------|
| `install`    | 安装一个角色。                            |
| `remove`     | 移除一个或多个角色。                        |
| `list`       | 显示已安装角色的名称和版本。                   |
| `info`       | 显示角色的信息。                           |
| `init`       | 生成一个新角色的骨架。                        |
| `import`     | 从 galaxy 网站导入一个角色。需要登录。           |

* 管理集合：

```bash
ansible-galaxy collection [import|init|install|login|remove|...]
```

| 子命令        | 功能                                   |
|--------------|----------------------------------------|
| `init`       | 生成一个新集合的骨架。                        |
| `install`    | 安装一个集合。                             |
| `list`       | 显示已安装集合的名称和版本。                     |

## Ansible 角色

Ansible 角色是一个促进 playbook 复用的单元。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)

### 安装有用的角色

为了突出使用角色的兴趣，我建议你使用 `alemorvan/patchmanagement` 角色，它将允许你在更新过程中执行许多任务（例如更新前或更新后），仅需几行代码。

你可以在角色的 github 仓库中查看代码[此处](https://github.com/alemorvan/patchmanagement)。

* 安装角色。这只需要一个命令：

```bash
ansible-galaxy role install alemorvan.patchmanagement
```

* 创建一个包含该角色的 playbook：

```bash
- name: Start a Patch Management
  hosts: ansible_clients
  vars:
    pm_before_update_tasks_file: custom_tasks/pm_before_update_tasks_file.yml
    pm_after_update_tasks_file: custom_tasks/pm_after_update_tasks_file.yml

  tasks:
    - name: "Include patchmanagement"
      include_role:
        name: "alemorvan.patchmanagement"
```

通过这个角色，你可以为你所有的清单或仅针对你的目标节点添加自己的任务。

让我们创建将在更新过程之前和之后运行的任务：

* 创建 `custom_tasks` 文件夹：

```bash
mkdir custom_tasks
```

* 创建 `custom_tasks/pm_before_update_tasks_file.yml`（随意更改此文件的名称和内容）

```bash
---
- name: sample task before the update process
  debug:
    msg: "This is a sample tasks, feel free to add your own test task"
```

* 创建 `custom_tasks/pm_after_update_tasks_file.yml`（随意更改此文件的名称和内容）

```bash
---
- name: sample task after the update process
  debug:
    msg: "This is a sample tasks, feel free to add your own test task"
```

然后启动你的第一个补丁管理：

```bash
ansible-playbook patchmanagement.yml

PLAY [Start a Patch Management] *************************************************************************

TASK [Gathering Facts] **********************************************************************************
ok: [192.168.1.11]

TASK [Include patchmanagement] **************************************************************************

TASK [alemorvan.patchmanagement : MAIN | Linux Patch Management Job] ************************************
ok: [192.168.1.11] => {
    "msg": "Start 192 patch management"
}

...

TASK [alemorvan.patchmanagement : sample task before the update process] ********************************
ok: [192.168.1.11] => {
    "msg": "This is a sample tasks, feel free to add your own test task"
}

...

TASK [alemorvan.patchmanagement : MAIN | We can now patch] **********************************************
included: /home/ansible/.ansible/roles/alemorvan.patchmanagement/tasks/patch.yml for 192.168.1.11

TASK [alemorvan.patchmanagement : PATCH | Tasks depends on distribution] ********************************
ok: [192.168.1.11] => {
    "ansible_distribution": "Rocky"
}

TASK [alemorvan.patchmanagement : PATCH | Include tasks for CentOS & RedHat tasks] **********************
included: /home/ansible/.ansible/roles/alemorvan.patchmanagement/tasks/linux_tasks/redhat_centos.yml for 192.168.1.11

TASK [alemorvan.patchmanagement : RHEL CENTOS | yum clean all] ******************************************
changed: [192.168.1.11]

TASK [alemorvan.patchmanagement : RHEL CENTOS | Ensure yum-utils is installed] **************************
ok: [192.168.1.11]

TASK [alemorvan.patchmanagement : RHEL CENTOS | Remove old kernels] *************************************
skipping: [192.168.1.11]

TASK [alemorvan.patchmanagement : RHEL CENTOS | Update rpm package with yum] ****************************
ok: [192.168.1.11]

TASK [alemorvan.patchmanagement : PATCH | Inlude tasks for Debian & Ubuntu tasks] ***********************
skipping: [192.168.1.11]

TASK [alemorvan.patchmanagement : MAIN | We can now reboot] *********************************************
included: /home/ansible/.ansible/roles/alemorvan.patchmanagement/tasks/reboot.yml for 192.168.1.11

TASK [alemorvan.patchmanagement : REBOOT | Reboot triggered] ********************************************
ok: [192.168.1.11]

TASK [alemorvan.patchmanagement : REBOOT | Ensure we are not in rescue mode] ****************************
ok: [192.168.1.11]

...

TASK [alemorvan.patchmanagement : FACTS | Insert fact file] *********************************************
ok: [192.168.1.11]

TASK [alemorvan.patchmanagement : FACTS | Save date of last PM] *****************************************
ok: [192.168.1.11]

...

TASK [alemorvan.patchmanagement : sample task after the update process] *********************************
ok: [192.168.1.11] => {
    "msg": "This is a sample tasks, feel free to add your own test task"
}

PLAY RECAP **********************************************************************************************
192.168.1.11               : ok=31   changed=1    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0  
```

对于如此复杂的过程来说，这是不是很容易？

这只是使用社区提供的角色可以完成的一个例子。查看 [galaxy.ansible.com](https://galaxy.ansible.com/) 来发现对你可能有用角色！

你也可以为自己的需求创建自己的角色，如果你愿意，可以在互联网上发布它们。这是我们将在下一章中简要介绍的内容。

### 角色开发介绍

角色骨架可以作为自定义角色开发的起点，可以通过 `ansible-galaxy` 命令生成：

```bash
$ ansible-galaxy role init rocky8
- Role rocky8 was created successfully
```

该命令将生成以下树结构来包含 `rocky8` 角色：

```bash
tree rocky8/
rocky8/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files
```

角色使你无需包含文件。无需在 playbook 中指定文件路径或 `include` 指令。你只需要指定一个任务，Ansible 负责包含。

角色的结构相当容易理解。

变量要么存储在 `vars/main.yml` 中（如果变量不应被覆盖），要么存储在 `default/main.yml` 中（如果你想留出从角色外部覆盖变量内容的可能性）。

你的代码所需的 handlers、files 和 templates 分别存储在 `handlers/main.yml`、`files` 和 `templates` 中。

剩下的就是为你的角色任务在 `tasks/main.yml` 中定义代码。

一旦这一切都运行良好，你就可以在你的 playbook 中使用这个角色。你将能够使用你的角色，而不用关心其任务的技术方面，同时通过变量自定义其操作。

### 实践：创建第一个简单的角色

让我们用一个"通用"角色来实现这一点，该角色将创建一个默认用户并安装软件包。此角色可以系统地应用于你的所有服务器。

#### 变量

我们将在所有服务器上创建一个 `rockstar` 用户。由于我们不希望此用户被覆盖，让我们在 `vars/main.yml` 中定义它：

```bash
---
rocky8_default_group:
  name: rockstar
  gid: 1100
rocky8_default_user:
  name: rockstar
  uid: 1100
  group: rockstar
```

我们现在可以在 `tasks/main.yml` 中使用这些变量，而无需任何包含。

```bash
---
- name: Create default group
  group:
    name: "{{ rocky8_default_group.name }}"
    gid: "{{ rocky8_default_group.gid }}"

- name: Create default user
  user:
    name: "{{ rocky8_default_user.name }}"
    uid: "{{ rocky8_default_user.uid }}"
    group: "{{ rocky8_default_user.group }}"
```

要测试你的新角色，让我们在角色所在目录中创建一个 `test-role.yml` playbook：

```bash
---
- name: Test my role
  hosts: localhost

  roles:

    - role: rocky8
      become: true
      become_user: root
```

然后启动它：

```bash
ansible-playbook test-role.yml

PLAY [Test my role] ************************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [localhost]

TASK [rocky8 : Create default group] *******************************************************************
changed: [localhost]

TASK [rocky8 : Create default user] ********************************************************************
changed: [localhost]

PLAY RECAP *********************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

恭喜！你现在可以用几行 playbook 创建很棒的东西了。

让我们看看默认变量的使用。

创建一个默认安装在你的服务器上的软件包列表和一个待卸载的空列表。编辑 `defaults/main.yml` 文件并添加这两个列表：

```bash
rocky8_default_packages:
  - tree
  - vim
rocky8_remove_packages: []
```

并在你的 `tasks/main.yml` 中使用它们：

```bash
- name: Install default packages (can be overridden)
  package:
    name: "{{ rocky8_default_packages }}"
    state: present

- name: "Uninstall default packages (can be overridden) {{ rocky8_remove_packages }}"
  package:
    name: "{{ rocky8_remove_packages }}"
    state: absent
```

使用之前创建的 playbook 测试你的角色：

```bash
ansible-playbook test-role.yml

PLAY [Test my role] ************************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [localhost]

TASK [rocky8 : Create default group] *******************************************************************
ok: [localhost]

TASK [rocky8 : Create default user] ********************************************************************
ok: [localhost]

TASK [rocky8 : Install default packages (can be overridden)] ********************************************
ok: [localhost]

TASK [rocky8 : Uninstall default packages (can be overridden) []] ***************************************
ok: [localhost]

PLAY RECAP *********************************************************************************************
localhost                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

现在你可以在 playbook 中覆盖 `rocky8_remove_packages`，并卸载例如 `cockpit`：

```bash
---
- name: Test my role
  hosts: localhost
  vars:
    rocky8_remove_packages:
      - cockpit

  roles:

    - role: rocky8
      become: true
      become_user: root
```

```bash
ansible-playbook test-role.yml

PLAY [Test my role] ************************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [localhost]

TASK [rocky8 : Create default group] *******************************************************************
ok: [localhost]

TASK [rocky8 : Create default user] ********************************************************************
ok: [localhost]

TASK [rocky8 : Install default packages (can be overridden)] ********************************************
ok: [localhost]

TASK [rocky8 : Uninstall default packages (can be overridden) ['cockpit']] ******************************
changed: [localhost]

PLAY RECAP *********************************************************************************************
localhost                  : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

显然，你可以无限地改进你的角色。想象一下，对于你的某个服务器，你需要一个在待卸载列表中的软件包。你可以创建例如一个可被覆盖的新列表，然后使用 jinja 的 `difference()` filter 从待卸载列表中移除那些需要保留的特定软件包。

```bash
- name: "Uninstall default packages (can be overridden) {{ rocky8_remove_packages }}"
  package:
    name: "{{ rocky8_remove_packages | difference(rocky8_specifics_packages) }}"
    state: absent
```

## Ansible 集合

集合是 Ansible 内容的分发格式，可以包括 playbook、角色、模块和插件。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)

安装或升级集合：

```bash
ansible-galaxy collection install namespace.collection [--upgrade]
```

然后你可以在模块名称或角色名称之前使用其命名空间和名称来使用新安装的集合：

```bash
- import_role:
    name: namespace.collection.rolename

- namespace.collection.modulename:
    option1: value
```

你可以在[此处](https://docs.ansible.com/ansible/latest/collections/index.html)找到集合索引。

让我们安装 `community.general` 集合：

```bash
ansible-galaxy collection install community.general
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/download/community-general-3.3.2.tar.gz to /home/ansible/.ansible/tmp/ansible-local-51384hsuhf3t5/tmpr_c9qrt1/community-general-3.3.2-f4q9u4dg
Installing 'community.general:3.3.2' to '/home/ansible/.ansible/collections/ansible_collections/community/general'
community.general:3.3.2 was installed successfully
```

我们现在可以使用新可用的模块 `yum_versionlock`：

```bash
- name: Start a Patch Management
  hosts: ansible_clients
  become: true
  become_user: root
  tasks:

    - name: Ensure yum-versionlock is installed
      package:
        name: python3-dnf-plugin-versionlock
        state: present

    - name: Prevent kernel from being updated
      community.general.yum_versionlock:
        state: present
        name: kernel
      register: locks

    - name: Display locks
      debug:
        var: locks.meta.packages                            
```

```bash
ansible-playbook versionlock.yml

PLAY [Start a Patch Management] *************************************************************************

TASK [Gathering Facts] **********************************************************************************
ok: [192.168.1.11]

TASK [Ensure yum-versionlock is installed] **************************************************************
changed: [192.168.1.11]

TASK [Prevent kernel from being updated] ****************************************************************
changed: [192.168.1.11]

TASK [Display locks] ************************************************************************************
ok: [192.168.1.11] => {
    "locks.meta.packages": [
        "kernel"
    ]
}

PLAY RECAP **********************************************************************************************
192.168.1.11               : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

### 创建你自己的集合

与角色一样，你可以使用 `ansible-galaxy` 命令创建自己的集合：

```bash
ansible-galaxy collection init rocky8.rockstarcollection
- Collection rocky8.rockstarcollection was created successfully
```

```bash
tree rocky8/rockstarcollection/
rocky8/rockstarcollection/
├── docs
├── galaxy.yml
├── plugins
│   └── README.md
├── README.md
└── roles
```

然后你可以将自己的插件或角色存储在这个新集合中。
