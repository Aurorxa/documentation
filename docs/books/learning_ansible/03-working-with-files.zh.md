---
title: 文件管理
---

# Ansible - 文件管理

在本章中，你将学习如何使用 Ansible 管理文件。

****

**目标**：在本章中你将学习：

:heavy_check_mark: 修改文件内容；
:heavy_check_mark: 将文件上传到目标服务器；
:heavy_check_mark: 从目标服务器检索文件。

:checkered_flag: **ansible**、**module**、**files**

**知识**：:star: :star:
**复杂度**：:star:

**阅读时间**：20 分钟

****

根据你的需求，你将需要使用不同的 Ansible 模块来修改系统配置文件。

## `ini_file` 模块

当你想要修改一个 INI 文件（位于 `[]` 之间的部分然后是 `key=value` 对）时，最简单的方法是使用 `ini_file` 模块。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/collections/community/general/ini_file_module.html)。

该模块需要：

* section（部分）的值
* option（选项）的名称
* 新的值

使用示例：

```bash
- name: change value on inifile
  community.general.ini_file:
    dest: /path/to/file.ini
    section: SECTIONNAME
    option: OPTIONNAME
    value: NEWVALUE
```

## `lineinfile` 模块

要确保某行存在于文件中，或者当文件中的单个行需要添加或修改时，使用 `lineinfile` 模块。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html)。

在这种情况下，将使用 regexp（正则表达式）来找到文件中要修改的行。

例如，要确保 `/etc/selinux/config` 文件中以 `SELINUX=` 开头的行包含值 `enforcing`：

```bash
- ansible.builtin.lineinfile:
    path: /etc/selinux/config
    regexp: '^SELINUX='
    line: 'SELINUX=enforcing'
```

## `copy` 模块

当文件需要从 Ansible 服务器复制到一个或多个主机时，最好使用 `copy` 模块。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)。

这里我们将 `myflile.conf` 从一个位置复制到另一个位置：

```bash
- ansible.builtin.copy:
    src: /data/ansible/sources/myfile.conf
    dest: /etc/myfile.conf
    owner: root
    group: root
    mode: 0644
```

## `fetch` 模块

当文件需要从远程服务器复制到本地服务器时，最好使用 `fetch` 模块。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fetch_module.html)。

该模块执行与 `copy` 模块相反的操作：

```bash
- ansible.builtin.fetch:
    src: /etc/myfile.conf
    dest: /data/ansible/backup/myfile-{{ inventory_hostname }}.conf
    flat: yes
```

## `template` 模块

Ansible 及其 `template` 模块使用 **Jinja2** 模板系统（<http://jinja.pocoo.org/docs/>）在目标主机上生成文件。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)。

例如：

```bash
- ansible.builtin.template:
    src: /data/ansible/templates/monfichier.j2
    dest: /etc/myfile.conf
    owner: root
    group: root
    mode: 0644
```

如果目标服务允许，可以添加一个验证步骤（例如 apache 使用 `apachectl -t` 命令）：

```bash
- template:
    src: /data/ansible/templates/vhost.j2
    dest: /etc/httpd/sites-available/vhost.conf
    owner: root
    group: root
    mode: 0644
    validate: '/usr/sbin/apachectl -t'
```

## `get_url` 模块

要将文件从网站或 ftp 上传到一个或多个主机，使用 `get_url` 模块：

```bash
- get_url:
    url: http://site.com/archive.zip
    dest: /tmp/archive.zip
    mode: 0640
    checksum: sha256:f772bd36185515581aa9a2e4b38fb97940ff28764900ba708e68286121770e9a
```

通过提供文件的校验和（checksum），如果文件已存在于目标位置且其校验和与提供的值匹配，则不会重新下载。
