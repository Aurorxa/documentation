---
title: 大规模基础设施
---

# Ansible - 大规模基础设施

在本章中，你将学习如何扩展你的配置管理系统。

****

**目标**：在本章中你将学习：

:heavy_check_mark: 为大型基础设施组织你的代码；
:heavy_check_mark: 将你的全部或部分配置管理应用到一个节点组；

:checkered_flag: **ansible**、**config management**、**scale**

**知识**：:star: :star: :star:
**复杂度**：:star: :star: :star: :star:

**阅读时间**：30 分钟

****

我们在前面的章节中已经看到了如何以角色的形式组织我们的代码，以及如何使用一些角色来管理更新（补丁管理）或代码部署。

那么配置管理呢？如何使用 Ansible 管理数十、数百甚至数千台虚拟机的配置？

云计算的到来稍微改变了传统方法。VM（虚拟机）在部署时进行配置。如果其配置不再符合要求，它将被销毁并替换为新的。

本章中介绍的配置管理系统组织方式将对这两种 IT 消费方式作出回应："一次性"使用或对服务器池进行定期的"重新配置"。

但是，请注意：使用 Ansible 确保服务器池的合规性需要改变工作习惯。不再可能手动修改服务管理器的配置，而不会在下一次 Ansible 运行时看到这些修改被覆盖。

!!! Note

    我们下面要设置的并不是 Ansible 最擅长的领域。像 Puppet 或 Salt 这样的技术会做得更好。让我们记住，Ansible 是自动化的瑞士军刀，并且是无代理的，这解释了性能上的差异。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/sample_setup.html)

## 变量存储

我们需要讨论的第一件事是数据与 Ansible 代码的分离。

随着代码变得越来越大、越来越复杂，修改其中包含的变量将变得越来越困难。

为了确保你的站点的可维护性，最重要的是正确分离变量和 Ansible 代码。

我们还没有在这里讨论过，但你应该知道 Ansible 可以根据被管理节点的清单名称或其所在的成员组，自动加载在特定文件夹中找到的变量。

Ansible 文档建议我们如下组织代码：

```bash
inventories/
   production/
      hosts               # 生产服务器的清单文件
      group_vars/
         group1.yml       # 在此处为特定组分配变量
         group2.yml
      host_vars/
         hostname1.yml    # 在此处为特定系统分配变量
         hostname2.yml
```

如果目标节点是 `group1` 中的 `hostname1`，则 `hostname1.yml` 和 `group1.yml` 文件中包含的变量将被自动加载。这是一种为所有角色在同一个地方存储所有数据的好方法。

通过这种方式，你服务器的清单文件成为其身份卡。它包含所有与你服务器的默认变量不同的变量。

从变量集中化的角度来看，必须在角色中组织变量的命名，例如以角色名称为变量名添加前缀。还建议使用扁平的变量名而不是字典。

例如，如果你想将 `sshd_config` 文件中的 `PermitRootLogin` 值作为变量，一个好的变量名可能是 `sshd_config_permitrootlogin`（而不是 `sshd.config.permitrootlogin`，后者也可能是一个好的变量名）。

## 关于 Ansible 标签（tags）

使用 Ansible 标签允许你执行或跳过代码中的一部分任务。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html)

例如，让我们修改我们的用户创建任务：

```bash
- name: add users
  user:
    name: "{{ item }}"
    state: present
    groups: "users"
  loop:
     - antoine
     - patrick
     - steven
     - xavier
  tags: users
```

你现在可以使用 `ansible-playbook` 的 `--tags` 选项仅运行带有 `users` 标签的任务：

```bash
ansible-playbook -i inventories/production/hosts --tags users site.yml
```

你也可以使用 `--skip-tags` 选项。

## 关于目录布局

让我们聚焦于一个 CMS（Content Management System，配置管理系统）正常运行所需的文件和目录的组织方案。

我们的起点将是 `site.yml` 文件。这个文件有点像是 CMS 的乐队指挥，因为它只会在需要时包含目标节点所需的角色：

```bash
---
- name: "Config Management for {{ target }}"
  hosts: "{{ target }}"

  roles:

    - role: roles/functionality1

    - role: roles/functionality2
```

当然，这些角色必须在 `site.yml` 文件同级目录的 `roles` 目录下创建。

我喜欢在 `vars/global_vars.yml` 中管理我的全局变量，即使我可以将它们存储在位于 `inventories/production/group_vars/all.yml` 的文件中：

```bash
---
- name: "Config Management for {{ target }}"
  hosts: "{{ target }}"
  vars_files:
    - vars/global_vars.yml
  roles:

    - role: roles/functionality1

    - role: roles/functionality2
```

我还喜欢保留禁用某个功能的可能性。因此，我以条件和默认值包含我的角色，如下所示：

```bash
---
- name: "Config Management for {{ target }}"
  hosts: "{{ target }}"
  vars_files:
    - vars/global_vars.yml
  roles:

    - role: roles/functionality1
      when:
        - enable_functionality1|default(true)

    - role: roles/functionality2
      when:
        - enable_functionality2|default(false)
```

不要忘记使用标签：

```bash
- name: "Config Management for {{ target }}"
  hosts: "{{ target }}"
  vars_files:
    - vars/global_vars.yml
  roles:

    - role: roles/functionality1
      when:
        - enable_functionality1|default(true)
      tags:
        - functionality1

    - role: roles/functionality2
      when:
        - enable_functionality2|default(false)
      tags:
        - functionality2
```

你应该得到类似以下的内容：

```bash
$ tree cms
cms
├── inventories
│   └── production
│       ├── group_vars
│       │   └── plateform.yml
│       ├── hosts
│       └── host_vars
│           ├── client1.yml
│           └── client2.yml
├── roles
│   ├── functionality1
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   └── functionality2
│       ├── defaults
│       │   └── main.yml
│       └── tasks
│           └── main.yml
├── site.yml
└── vars
    └── global_vars.yml
```

!!! Note

    你可以自由地在集合中开发你的角色

## 测试

让我们启动 playbook 并运行一些测试：

```bash
$ ansible-playbook -i inventories/production/hosts -e "target=client1" site.yml

PLAY [Config Management for client1] ****************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [client1]

TASK [roles/functionality1 : Task in functionality 1] *********************************************************
ok: [client1] => {
    "msg": "You are in functionality 1"
}

TASK [roles/functionality2 : Task in functionality 2] *********************************************************
skipping: [client1]

PLAY RECAP ******************************************************************************************************
client1                    : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```

如你所见，默认情况下只运行 `functionality1` 角色的任务。

让我们在清单中为我们的目标节点激活 `functionality2` 并重新运行 playbook：

```bash
$ vim inventories/production/host_vars/client1.yml
---
enable_functionality2: true
```

```bash
$ ansible-playbook -i inventories/production/hosts -e "target=client1" site.yml

PLAY [Config Management for client1] ****************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [client1]

TASK [roles/functionality1 : Task in functionality 1] *********************************************************
ok: [client1] => {
    "msg": "You are in functionality 1"
}

TASK [roles/functionality2 : Task in functionality 2] *********************************************************
ok: [client1] => {
    "msg": "You are in functionality 2"
}

PLAY RECAP ******************************************************************************************************
client1                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

尝试仅应用 `functionality2`：

```bash
$ ansible-playbook -i inventories/production/hosts -e "target=client1" --tags functionality2 site.yml

PLAY [Config Management for client1] ****************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [client1]

TASK [roles/functionality2 : Task in functionality 2] *********************************************************
ok: [client1] => {
    "msg": "You are in functionality 2"
}

PLAY RECAP ******************************************************************************************************
client1                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

让我们在整个清单上运行：

```bash
$ ansible-playbook -i inventories/production/hosts -e "target=plateform" site.yml

PLAY [Config Management for plateform] **************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [client1]
ok: [client2]

TASK [roles/functionality1 : Task in functionality 1] *********************************************************
ok: [client1] => {
    "msg": "You are in functionality 1"
}
ok: [client2] => {
    "msg": "You are in functionality 1"
}

TASK [roles/functionality2 : Task in functionality 2] *********************************************************
ok: [client1] => {
    "msg": "You are in functionality 2"
}
skipping: [client2]

PLAY RECAP ******************************************************************************************************
client1                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
client2                    : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```

如你所见，`functionality2` 仅在 `client1` 上运行。

## 收益

通过遵循 Ansible 文档中给出的建议，你将快速获得一个：

* 易于维护的源代码，即使其中包含大量角色
* 一个相对快速、可重复的合规系统，你可以部分或完全应用
* 可以按情况、按服务器进行适配
* 你的信息系统的特定内容与代码分离，易于审计，并集中在你的配置管理的清单文件中
