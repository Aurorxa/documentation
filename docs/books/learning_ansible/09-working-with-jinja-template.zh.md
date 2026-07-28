---
title: 在 Ansible 中使用 Jinja 模板
author: Srinivas Nishant Viswanadha
contributors: Steven Spencer, Antoine Le Morvan, Ganna Zhyrnova
---

## 简介

Ansible 提供了一种强大而简单的方式，通过内置的 `template` 模块来使用 Jinja 模板管理配置。本文档探讨了在 Ansible 中使用 Jinja 模板的两种基本方式：

- 向配置文件添加变量
- 通过循环和复杂的数据结构构建完整的文件。

## 向配置文件添加变量

### 步骤 1：创建 Jinja 模板

创建一个 Jinja 模板文件，例如 `sshd_config.j2`，其中包含变量的占位符：

```jinja
# /path/to/sshd_config.j2

Port {{ ssh_port }}
PermitRootLogin {{ permit_root_login }}
# Add more variables as needed
```

### 步骤 2：使用 Ansible template 模块

在你的 Ansible playbook 中，使用 `template` 模块以特定值渲染 Jinja 模板：

```yaml
---
- name: Generate sshd_config
  hosts: your_target_hosts
  tasks:
    - name: Template sshd_config
      template:
        src: /path/to/sshd_config.j2
        dest: /etc/ssh/sshd_config
      vars:
        ssh_port: 22
        permit_root_login: no
      # Add more variables as needed
```

### 步骤 3：应用配置更改

执行 Ansible playbook 以将更改应用于目标主机：

```bash
ansible-playbook your_playbook.yml
```

此步骤确保配置更改在你整个基础设施中一致地应用。

## 使用循环和复杂数据结构构建完整文件

### 步骤 1：增强 Jinja 模板

扩展你的 Jinja 模板以处理循环和复杂的数据结构。以下是配置一个具有多个组件的假设应用程序的示例：

```jinja
# /path/to/app_config.j2

{% for component in components %}
[{{ component.name }}]
    Path = {{ component.path }}
    Port = {{ component.port }}
    # Add more configurations as needed
{% endfor %}
```

### 步骤 2：集成 Ansible template 模块

在你的 Ansible playbook 中，集成 `template` 模块以生成一个完整的配置文件：

```yaml
---
- name: Generate Application Configuration
  hosts: your_target_hosts
  vars:
    components:
      - name: web_server
        path: /var/www/html
        port: 80
      - name: database
        path: /var/lib/db
        port: 3306
      # Add more components as needed
  tasks:
    - name: Template Application Configuration
      template:
        src: /path/to/app_config.j2
        dest: /etc/app/config.ini
```

执行 Ansible playbook 以将更改应用于目标主机：

```bash
ansible-playbook your_playbook.yml
```

此步骤确保配置更改在你整个基础设施中一致地应用。

Ansible 的 `template` 模块允许你在 playbook 执行期间使用 Jinja 模板动态生成配置文件。此模块允许你将配置逻辑与数据分离，使你的 Ansible playbook 更加灵活和可维护。

### 关键特性

1. **模板渲染：**
   - 该模块渲染 Jinja 模板以创建具有动态内容的配置文件。
   - 在 playbook 或清单中定义的变量可以注入到模板中，从而实现动态配置。

2. **Jinja2 的使用：**
   - `template` 模块利用 Jinja2 模板引擎，提供强大的功能，如条件、循环和过滤器，用于高级模板操作。

3. **源路径和目标路径：**
   - 指定源 Jinja 模板文件和生成的配置文件的目标路径。

4. **变量传递：**
   - 变量可以直接在 playbook 任务中传递，也可以从外部文件加载，允许灵活和动态的配置生成。

5. **幂等执行：**
   - template 模块支持幂等执行，确保只有在检测到更改时才应用模板。

### 示例 playbook 片段

```yaml
---
- name: Generate Configuration File
  hosts: your_target_hosts
  tasks:
    - name: Template Configuration File
      template:
        src: /path/to/template.j2
        dest: /etc/config/config_file
      vars:
        variable1: value1
        variable2: value2
```

### 使用场景

1. **配置管理：**
   - 非常适合通过基于特定参数动态生成文件来管理系统配置。

2. **应用程序设置：**
   - 用于创建具有不同设置的应用程序特定配置文件。

3. **基础设施即代码：**
   - 通过允许基于变量动态调整配置，促进了基础设施即代码（Infrastructure as Code）实践。

### 最佳实践

1. **关注点分离：**
   - 将实际的配置逻辑保留在 Jinja 模板中，将其与 playbook 的主要结构分离。

2. **版本控制：**
   - 将 Jinja 模板存储在版本控制的仓库中，以便更好地跟踪和协作。

3. **可测试性：**
   - 独立测试模板，确保它们产生预期的配置输出。

通过利用 `template` 模块，Ansible 用户可以增强配置任务的可管理性和灵活性，促进更简化和高效的系统与应用程序设置方法。

### 参考资料

[Ansible Template 模块文档](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)。
