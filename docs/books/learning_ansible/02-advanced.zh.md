---
title: Ansible 中级
---

# Ansible 中级

在本章中，你将继续学习如何使用 Ansible。

****

**目标**：在本章中你将学习：

:heavy_check_mark: 使用变量；
:heavy_check_mark: 使用循环；
:heavy_check_mark: 管理状态更改并对其做出反应；
:heavy_check_mark: 管理异步任务。

:checkered_flag: **ansible**、**module**、**playbook**

**知识**：:star: :star: :star:
**复杂度**：:star: :star:

**阅读时间**：30 分钟

****

在上一章中，你学习了如何安装 Ansible，在命令行上使用它，并编写 playbook 以促进代码的复用。

在本章中，我们可以开始发现如何使用 Ansible 的更高级概念以及你将经常使用的一些有趣任务。

## 变量

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)。

在 Ansible 下，有不同类型的原始变量：

* 字符串（strings），
* 整数（integers），
* 布尔值（booleans）。

这些变量可以组织为：

* 字典（dictionaries），
* 列表（lists）。

变量可以在不同的地方定义，例如 playbook、角色或命令行。

例如，在 playbook 中：

```bash
---
- hosts: apache1
  vars:
    port_http: 80
    service:
      debian: apache2
      rhel: httpd
```

或在命令行中：

```bash
ansible-playbook deploy-http.yml --extra-vars "service=httpd"
```

一旦定义，可以通过在双花括号之间调用变量来使用它：

* `{{ port_http }}` 用于简单值，
* `{{ service['rhel'] }}` 或 `{{ service.rhel }}` 用于字典。

例如：

```bash
- name: make sure apache is started
  ansible.builtin.systemd:
    name: "{{ service['rhel'] }}"
    state: started
```

当然，也可以访问 Ansible 的全局变量（**事实**）（OS 类型、IP 地址、VM 名称等）。

### 外部化变量

变量可以包含在 playbook 外部的文件中，在这种情况下，必须使用 `vars_files` 指令在 playbook 中定义此文件：

```bash
---
- hosts: apache1
  vars_files:
    - myvariables.yml
```

`myvariables.yml` 文件：

```bash
---
port_http: 80
ansible.builtin.systemd::
  debian: apache2
  rhel: httpd
```

也可以使用 `include_vars` 模块动态添加：

```bash
- name: Include secrets.
  ansible.builtin.include_vars:
    file: vault.yml
```

### 显示变量

要显示变量，你必须激活 `debug` 模块，如下所示：

```bash
- ansible.builtin.debug:
    var: service['debian']
```

你也可以在文本中使用变量：

```bash
- ansible.builtin.debug:
    msg: "Print a variable in a message : {{ service['debian'] }}"
```

### 保存任务的返回值

要保存任务的返回值并在以后能够访问它，你必须使用任务本身中的 `register` 关键字。

使用存储的变量：

```bash
- name: /home content
  shell: ls /home
  register: homes

- name: Print the first directory name
  ansible.builtin.debug:
    var: homes.stdout_lines[0]

- name: Print the first directory name
  ansible.builtin.debug:
    var: homes.stdout_lines[1]
```

!!! Note

    变量 `homes.stdout_lines` 是一个字符串类型的变量列表，一种我们尚未遇到的变量组织方式。

构成存储变量的字符串可以通过 `stdout` 值访问（这允许你做类似 `homes.stdout.find("core") != -1` 的事情），利用循环（见 `loop`）利用它们，或者简单地通过它们的索引来访问，如前面的示例所示。

### 练习：

* 编写一个 playbook `play-vars.yml`，使用全局变量打印目标系统的发行版名称和主要版本。

* 编写一个使用以下字典的 playbook 来显示将要安装的服务：

```bash
service:
  web:
    name: apache
    rpm: httpd
  db:
    name: mariadb
    rpm: mariadb-server
```

默认类型应为 "web"。

* 使用命令行覆盖 `type` 变量

* 将变量外部化到 `vars.yml` 文件中

## 循环管理

循环（loop）允许你对列表、哈希或字典等进行任务迭代。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)。

一个简单的使用示例，创建 4 个用户：

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
```

在循环的每次迭代中，使用的列表的值存储在 `item` 变量中，可在循环代码中访问。

当然，列表可以在外部文件中定义：

```bash
users:
  - antoine
  - patrick
  - steven
  - xavier
```

并在任务中像这样使用（在包含 vars 文件之后）：

```bash
- name: add users
  user:
    name: "{{ item }}"
    state: present
    groups: "users"
  loop: "{{ users }}"
```

我们可以使用在学习存储变量时看到的示例来改进它。使用存储的变量：

```bash
- name: /home content
  shell: ls /home
  register: homes

- name: Print the directories name
  ansible.builtin.debug:
    msg: "Directory => {{ item }}"
  loop: "{{ homes.stdout_lines }}"
```

字典也可以在循环中使用。

在这种情况下，你必须使用 **jinja filter**（jinja 是 Ansible 使用的模板引擎）将字典转换为条目：`| dict2items`。

在循环中，可以使用 `item.key`（对应字典键）和 `item.value`（对应键的值）。

让我们通过一个具体的例子来了解这一点，展示系统用户的管理：

```bash
---
- hosts: rocky8
  become: true
  become_user: root
  vars:
    users:
      antoine:
        group: users
        state: present
      steven:
        group: users
        state: absent

  tasks:

  - name: Manage users
    user:
      name: "{{ item.key }}"
      group: "{{ item.value.group }}"
      state: "{{ item.value.state }}"
    loop: "{{ users | dict2items }}"
```

!!! Note

    循环可以用于很多事情。当你对 Ansible 的使用促使你更复杂地使用它们时，你将发现它们提供的可能性。

### 练习：

* 使用循环显示上一个练习中的 `service` 变量的内容。

!!! Note

    你需要使用 jinja filter `list` 将你的 `service` 变量（它是一个字典）转换为列表，如下所示：

    ```
    {{ service.values() | list }}
    ```

## 条件语句

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)。

`when` 语句在很多情况下非常有用，例如不在某些类型的服务器上执行某些操作，如果文件或用户不存在等。

!!! Note

    在 `when` 语句后面，变量不需要双花括号（它们实际上是 Jinja2 表达式……）。

```bash
- name: "Reboot only Debian servers"
  reboot:
  when: ansible_os_family == "Debian"
```

条件可以用括号分组：

```bash
- name: "Reboot only CentOS version 6 and Debian version 7"
  reboot:
  when: (ansible_distribution == "CentOS" and ansible_distribution_major_version == "6") or
        (ansible_distribution == "Debian" and ansible_distribution_major_version == "7")
```

对应于逻辑 AND 的条件可以作为列表提供：

```bash
- name: "Reboot only CentOS version 6"
  reboot:
  when:
    - ansible_distribution == "CentOS"
    - ansible_distribution_major_version == "6"
```

你可以测试布尔值的真值并验证其为真：

```bash
- name: check if directory exists
  stat:
    path: /home/ansible
  register: directory

- ansible.builtin.debug:
    var: directory

- ansible.builtin.debug:
    msg: The directory exists
  when:
    - directory.stat.exists
    - directory.stat.isdir
```

你也可以测试其为假：

```bash
when:
  - file.stat.exists
  - not file.stat.isdir
```

你可能需要测试一个变量是否存在以避免执行错误：

```bash
when: myboolean is defined and myboolean
```

### 练习：

* 仅当 `type` 等于 `web` 时打印 `service.web` 的值。

## 管理更改：`handlers`（处理器）

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html)。

当发生更改时，handlers 允许启动操作，如重启服务。

一个模块是幂等的，playbook 可以检测到远程系统上存在重大更改，从而触发响应该更改的操作。通知在 playbook 任务块的末尾发送，反应操作将只触发一次，即使多个任务发送相同的通知。

![Handlers](images/handlers.png)

例如，多个任务可能指示 `httpd` 服务需要由于其配置文件的更改而重新启动。但是，该服务只会重新启动一次，以避免多次不必要的启动。

```bash
- name: template configuration file
  template:
    src: template-site.j2
    dest: /etc/httpd/sites-availables/test-site.conf
  notify:
     - restart memcached
     - restart httpd
```

handler 是一种由唯一全局名称引用的任务类型：

* 一个或多个通知器激活它。
* 它不会立即启动，而是等待所有任务完成后才运行。

处理器的示例：

```bash
handlers:

  - name: restart memcached
    systemd:
      name: memcached
      state: restarted

  - name: restart httpd
    systemd:
      name: httpd
      state: restarted
```

自 Ansible 2.2 版本以来，handlers 也可以直接监听：

```bash
handlers:

  - name: restart memcached
    systemd:
      name: memcached
      state: restarted
    listen: "web services restart"

  - name: restart apache
    systemd:
      name: apache
      state: restarted
    listen: "web services restart"

tasks:
    - name: restart everything
      command: echo "this task will restart the web services"
      notify: "web services restart"
```

## 异步任务

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/playbooks_async.html)。

默认情况下，与主机的 SSH 连接在在所有节点上执行各种 playbook 任务时保持开启状态。

这可能导致一些问题，特别是：

* 如果任务的执行时间超过 SSH 连接超时时间
* 如果在操作过程中连接中断（例如，服务器重启）

在这种情况下，你将需要切换到异步模式，并指定最大执行时间和轮询主机状态的频率（默认 10 秒）。

通过指定轮询值为 0，Ansible 将执行任务并继续，而不关心结果。

以下是一个使用异步任务的示例，允许你重启服务器并等待端口 22 再次可达：

```bash
# Wait 2s and launch the reboot
- name: Reboot system
  shell: sleep 2 && shutdown -r now "Ansible reboot triggered"
  async: 1
  poll: 0
  ignore_errors: true
  become: true
  changed_when: False

  # Wait the server is available
  - name: Waiting for server to restart (10 mins max)
    wait_for:
      host: "{{ inventory_hostname }}"
      port: 22
      delay: 30
      state: started
      timeout: 600
    delegate_to: localhost
```

你也可以决定启动一个长时间运行的任务并忘记它（fire and forget），因为在 playbook 中执行不重要。

## 练习结果

* 编写一个 playbook `play-vars.yml`，使用全局变量打印目标系统的发行版名称和主要版本。

```bash
---
- hosts: ansible_clients

  tasks:

    - name: Print globales variables
      debug:
        msg: "The distribution is {{ ansible_distribution }} version {{ ansible_distribution_major_version }}"
```

```bash
$ ansible-playbook play-vars.yml

PLAY [ansible_clients] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.1.11]

TASK [Print globales variables] ************************************************************************
ok: [192.168.1.11] => {
    "msg": "The distribution is Rocky version 8"
}

PLAY RECAP *********************************************************************************************
192.168.1.11               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

* 编写一个使用以下字典的 playbook 来显示将要安装的服务：

```bash
service:
  web:
    name: apache
    rpm: httpd
  db:
    name: mariadb
    rpm: mariadb-server
```

默认类型应为 "web"。

```bash
---
- hosts: ansible_clients
  vars:
    type: web
    service:
      web:
        name: apache
        rpm: httpd
      db:
        name: mariadb
        rpm: mariadb-server

  tasks:

    - name: Print a specific entry of a dictionary
      debug:
        msg: "The {{ service[type]['name'] }} will be installed with the packages {{ service[type].rpm }}"
```

```bash
$ ansible-playbook display-dict.yml

PLAY [ansible_clients] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.1.11]

TASK [Print a specific entry of a dictionnaire] ********************************************************
ok: [192.168.1.11] => {
    "msg": "The apache will be installed with the packages httpd"
}

PLAY RECAP *********************************************************************************************
192.168.1.11               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

* 使用命令行覆盖 `type` 变量：

```bash
ansible-playbook --extra-vars "type=db" display-dict.yml

PLAY [ansible_clients] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.1.11]

TASK [Print a specific entry of a dictionary] ********************************************************
ok: [192.168.1.11] => {
    "msg": "The mariadb will be installed with the packages mariadb-server"
}

PLAY RECAP *********************************************************************************************
192.168.1.11               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

* 将变量外部化到 `vars.yml` 文件中

```bash
type: web
service:
  web:
    name: apache
    rpm: httpd
  db:
    name: mariadb
    rpm: mariadb-server
```

```bash
---
- hosts: ansible_clients
  vars_files:
    - vars.yml

  tasks:

    - name: Print a specific entry of a dictionary
      debug:
        msg: "The {{ service[type]['name'] }} will be installed with the packages {{ service[type].rpm }}"
```

* 使用循环显示上一个练习中的 `service` 变量的内容。

!!! Note

    你需要使用 jinja filters `dict2items` 或 `list` 将你的 `service` 变量（它是一个字典）转换为条目或列表，如下所示：

    ```
    {{ service | dict2items }}
    ```

    ```
    {{ service.values() | list }}
    ```

使用 `dict2items`：

```bash
---
- hosts: ansible_clients
  vars_files:
    - vars.yml

  tasks:

    - name: Print a dictionary variable with a loop
      debug:
        msg: "{{item.key }} | The {{ item.value.name }} will be installed with the packages {{ item.value.rpm }}"
      loop: "{{ service | dict2items }}"              
```

```bash
$ ansible-playbook display-dict.yml

PLAY [ansible_clients] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.1.11]

TASK [Print a dictionary variable with a loop] ********************************************************
ok: [192.168.1.11] => (item={'key': 'web', 'value': {'name': 'apache', 'rpm': 'httpd'}}) => {
    "msg": "web | The apache will be installed with the packages httpd"
}
ok: [192.168.1.11] => (item={'key': 'db', 'value': {'name': 'mariadb', 'rpm': 'mariadb-server'}}) => {
    "msg": "db | The mariadb will be installed with the packages mariadb-server"
}

PLAY RECAP *********************************************************************************************
192.168.1.11               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

使用 `list`：

```bash
---
- hosts: ansible_clients
  vars_files:
    - vars.yml

  tasks:

    - name: Print a dictionary variable with a loop
      debug:
        msg: "The {{ item.name }} will be installed with the packages {{ item.rpm }}"
      loop: "{{ service.values() | list}}"
~                                                 
```

```bash
$ ansible-playbook display-dict.yml

PLAY [ansible_clients] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.1.11]

TASK [Print a dictionary variable with a loop] ********************************************************
ok: [192.168.1.11] => (item={'name': 'apache', 'rpm': 'httpd'}) => {
    "msg": "The apache will be installed with the packages httpd"
}
ok: [192.168.1.11] => (item={'name': 'mariadb', 'rpm': 'mariadb-server'}) => {
    "msg": "The mariadb will be installed with the packages mariadb-server"
}

PLAY RECAP *********************************************************************************************
192.168.1.11               : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

* 仅当 `type` 等于 `web` 时打印 `service.web` 的值。

```bash
---
- hosts: ansible_clients
  vars_files:
    - vars.yml

  tasks:

    - name: Print a dictionary variable
      debug:
        msg: "The {{ service.web.name }} will be installed with the packages {{ service.web.rpm }}"
      when: type == "web"


    - name: Print a dictionary variable
      debug:
        msg: "The {{ service.db.name }} will be installed with the packages {{ service.db.rpm }}"
      when: type == "db"
```

```bash
$ ansible-playbook display-dict.yml

PLAY [ansible_clients] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.1.11]

TASK [Print a dictionary variable] ********************************************************************
ok: [192.168.1.11] => {
    "msg": "The apache will be installed with the packages httpd"
}

TASK [Print a dictionary variable] ********************************************************************
skipping: [192.168.1.11]

PLAY RECAP *********************************************************************************************
192.168.1.11               : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

$ ansible-playbook --extra-vars "type=db" display-dict.yml

PLAY [ansible_clients] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.1.11]

TASK [Print a dictionary variable] ********************************************************************
skipping: [192.168.1.11]

TASK [Print a dictionary variable] ********************************************************************
ok: [192.168.1.11] => {
    "msg": "The mariadb will be installed with the packages mariadb-server"
}

PLAY RECAP *********************************************************************************************
192.168.1.11               : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```
