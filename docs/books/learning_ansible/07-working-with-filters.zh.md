---
title: 使用过滤器
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
---

# Ansible - 使用过滤器

在本章中，你将学习如何使用 jinja 过滤器转换数据。

****

**目标**：在本章中你将学习：

:heavy_check_mark: 转换数据结构，如字典或列表；
:heavy_check_mark: 转换变量。

:checkered_flag: **ansible**、**jinja**、**filters**

**知识**：:star: :star: :star:
**复杂度**：:star: :star: :star: :star:

**阅读时间**：20 分钟

****

在之前的章节中，我们已经有机会使用 jinja 过滤器。

这些用 python 编写的过滤器允许我们操作和转换我们的 ansible 变量。

!!! Note

    更多信息可以在[此处找到](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html)。

在本章中，我们将使用以下 playbook 来测试所介绍的不同过滤器：

```bash
- name: Manipulating the data
  hosts: localhost
  gather_facts: false
  vars:
    zero: 0
    zero_string: "0"
    non_zero: 4
    true_booleen: True
    true_non_booleen: "True"
    false_boolean: False
    false_non_boolean: "False"
    whatever: "It's false!"
    user_name: antoine
    my_dictionary:
      key1: value1
      key2: value2
    my_simple_list:
      - value_list_1
      - value_list_2
      - value_list_3
    my_simple_list_2:
      - value_list_3
      - value_list_4
      - value_list_5
    my_list:
      - element: element1
        value: value1
      - element: element2
        value: value2

  tasks:
    - name: Print an integer
      debug:
        var: zero
```

!!! Note

    以下是你最可能遇到或需要的过滤器的非详尽列表。
    幸运的是，还有许多其他过滤器。你甚至可以编写自己的过滤器！

playbook 将按以下方式运行：

```bash
ansible-playbook play-filter.yml
```

## 转换数据

数据可以从一种类型转换为另一种类型。

要了解数据的类型（python 语言中的类型），你必须使用 `type_debug` 过滤器。

示例：

```bash
- name: Display the type of a variable
  debug:
    var: true_boolean|type_debug
```

这将给出：

```bash
TASK [Display the type of a variable] ******************************************************************
ok: [localhost] => {
    "true_boolean|type_debug": "bool"
}
```

可以将整数转换为字符串：

```bash
- name: Transforming a variable type
  debug:
    var: zero|string
```

```bash
TASK [Transforming a variable type] ***************************************************************
ok: [localhost] => {
    "zero|string": "0"
}
```

将字符串转换为整数：

```bash
- name: Transforming a variable type
  debug:
    var: zero_string|int
```

或将变量转换为布尔值：

```bash
- name: Display an integer as a boolean
  debug:
    var: non_zero | bool

- name: Display a string as a boolean
  debug:
    var: true_non_boolean | bool

- name: Display a string as a boolean
  debug:
    var: false_non_boolean | bool

- name: Display a string as a boolean
  debug:
    var: whatever | bool

```

可以将字符串转换为大写或小写：

```bash
- name: Lowercase a string of characters
  debug:
    var: whatever | lower

- name: Upercase a string of characters
  debug:
    var: whatever | upper
```

这将给出：

```bash
TASK [Lowercase a string of characters] *****************************************************
ok: [localhost] => {
    "whatever | lower": "it's false!"
}

TASK [Upercase a string of characters] *****************************************************
ok: [localhost] => {
    "whatever | upper": "IT'S FALSE!"
}
```

`replace` 过滤器允许将字符替换为其他字符。

这里我们移除空格甚至替换一个单词：

```bash
- name: Replace a character in a string
  debug:
    var: whatever | replace(" ", "")

- name: Replace a word in a string
  debug:
    var: whatever | replace("false", "true")
```

这将给出：

```bash
TASK [Replace a character in a string] *****************************************************
ok: [localhost] => {
    "whatever | replace(\" \", \"\")": "It'sfalse!"
}

TASK [Replace a word in a string] *****************************************************
ok: [localhost] => {
    "whatever | replace(\"false\", \"true\")": "It's true !"
}
```

`split` 过滤器根据一个字符将字符串分割成列表：

```bash
- name: Cutting a string of characters
  debug:
    var: whatever | split(" ", "")
```

```bash
TASK [Cutting a string of characters] *****************************************************
ok: [localhost] => {
    "whatever | split(\" \")": [
        "It's",
        "false!"
    ]
}
```

## 连接列表的元素

经常需要将不同的元素连接到一个字符串中。
我们可以指定一个字符或字符串插入到每个元素之间。

```bash
- name: Joining elements of a list
  debug:
    var: my_simple_list|join(",")

- name: Joining elements of a list
  debug:
    var: my_simple_list|join(" | ")
```

这将给出：

```bash
TASK [Joining elements of a list] *****************************************************************
ok: [localhost] => {
    "my_simple_list|join(\",\")": "value_list_1,value_list_2,value_list_3"
}

TASK [Joining elements of a list] *****************************************************************
ok: [localhost] => {
    "my_simple_list|join(\" | \")": "value_list_1 | value_list_2 | value_list_3"
}

```

## 将字典转换为列表（反之亦然）

过滤器 `dict2items` 和 `itemstodict`，实现起来稍微复杂一些，但在循环中经常使用。

请注意，可以指定转换时使用的键名和值名。

```bash
- name: Display a dictionary
  debug:
    var: my_dictionary

- name: Transforming a dictionary into a list
  debug:
    var: my_dictionary | dict2items

- name: Transforming a dictionary into a list
  debug:
    var: my_dictionary | dict2items(key_name='key', value_name='value')

- name: Transforming a list into a dictionary
  debug:
    var: my_list | items2dict(key_name='element', value_name='value')
```

```bash
TASK [Display a dictionary] *************************************************************************
ok: [localhost] => {
    "my_dictionary": {
        "key1": "value1",
        "key2": "value2"
    }
}

TASK [Transforming a dictionary into a list] *************************************************************
ok: [localhost] => {
    "my_dictionary | dict2items": [
        {
            "key": "key1",
            "value": "value1"
        },
        {
            "key": "key2",
            "value": "value2"
        }
    ]
}

TASK [Transforming a dictionary into a list] *************************************************************
ok: [localhost] => {
    "my_dictionary | dict2items (key_name = 'key', value_name = 'value')": [
        {
            "key": "key1",
            "value": "value1"
        },
        {
            "key": "key2",
            "value": "value2"
        }
    ]
}

TASK [Transforming a list into a dictionary] ************************************************************
ok: [localhost] => {
    "my_list | items2dict(key_name='element', value_name='value')": {
        "element1": "value1",
        "element2": "value2"
    }
}
```

## 处理列表

可以合并或过滤一个或多个列表中的数据：

```bash
- name: Merger of two lists
  debug:
    var: my_simple_list | union(my_simple_list_2)
```

```bash
ok: [localhost] => {
    "my_simple_list | union(my_simple_list_2)": [
        "value_list_1",
        "value_list_2",
        "value_list_3",
        "value_list_4",
        "value_list_5"
    ]
}
```

仅保留两个列表的交集（两个列表中都存在的值）：

```bash
- name: Merger of two lists
  debug:
    var: my_simple_list | intersect(my_simple_list_2)
```

```bash
TASK [Merger of two lists] *******************************************************************************
ok: [localhost] => {
    "my_simple_list | intersect(my_simple_list_2)": [
        "value_list_3"
    ]
}
```

或者相反，仅保留差集（在第二个列表中不存在的值）：

```bash
- name: Merger of two lists
  debug:
    var: my_simple_list | difference(my_simple_list_2)
```

```bash
TASK [Merger of two lists] *******************************************************************************
ok: [localhost] => {
    "my_simple_list | difference(my_simple_list_2)": [
        "value_list_1",
        "value_list_2",
    ]
}
```

如果你的列表包含非唯一值，也可以使用 `unique` 过滤器过滤它们。

```bash
- name: Unique value in a list
  debug:
    var: my_simple_list | unique
```

## json/yaml 转换

你可能需要导入 json 数据（例如来自 API）或将数据导出为 yaml 或 json。

```bash
- name: Display a variable in yaml
  debug:
    var: my_list | to_nice_yaml(indent=4)

- name: Display a variable in json
  debug:
    var: my_list | to_nice_json(indent=4)
```

```bash
TASK [Display a variable in yaml] ********************************************************************
ok: [localhost] => {
    "my_list | to_nice_yaml(indent=4)": "-   element: element1\n    value: value1\n-   element: element2\n    value: value2\n"
}

TASK [Display a variable in json] ********************************************************************
ok: [localhost] => {
    "my_list | to_nice_json(indent=4)": "[\n    {\n        \"element\": \"element1\",\n        \"value\": \"value1\"\n    },\n    {\n        \"element\": \"element2\",\n        \"value\": \"value2\"\n    }\n]"
}
```

## 默认值、可选变量、保护变量

如果你不为变量提供默认值或者不保护它们，你很快会在 playbook 执行时遇到错误。

使用 `default` 过滤器，如果变量不存在，则可用另一个值替换其值：

```bash
- name: Default value
  debug:
    var: variablethatdoesnotexists | default(whatever)
```

```bash
TASK [Default value] ********************************************************************************
ok: [localhost] => {
    "variablethatdoesnotexists | default(whatever)": "It's false!"
}
```

注意撇号 `'` 的存在，如果你使用 `shell` 模块，应该保护它：

```bash
- name: Default value
  debug:
    var: variablethatdoesnotexists | default(whatever| quote)
```

```bash
TASK [Default value] ********************************************************************************
ok: [localhost] => {
    "variablethatdoesnotexists | default(whatever|quote)": "'It'\"'\"'s false!'"
}
```

最后，如果模块中的可选变量不存在，可以使用 `default` 过滤器中的 `omit` 关键字忽略它，这将使你在运行时免于错误。

```bash
- name: Add a new user
  ansible.builtin.user:
    name: "{{ user_name }}"
    comment: "{{ user_comment | default(omit) }}"
```

## 根据另一个值关联一个值（`ternary`）

有时你需要使用条件为变量赋值，在这种情况下通常需要通过 `set_fact` 步骤。

使用 `ternary` 过滤器可以避免这种情况：

```bash
- name: Default value
  debug:
    var: (user_name == 'antoine') | ternary('admin', 'normal_user')
```

```bash
TASK [Default value] ********************************************************************************
ok: [localhost] => {
    "(user_name == 'antoine') | ternary('admin', 'normal_user')": "admin"
}
```

## 其他一些过滤器

* `{{ 10000 | random }}`：如其名所示，给出一个随机值。
* `{{ my_simple_list | first }}`：提取列表的第一个元素。
* `{{ my_simple_list | length }}`：给出长度（列表或字符串）。
* `{{ ip_list | ansible.netcommon.ipv4 }}`：仅显示 v4 IP。如果没有深入了解，如果需要，有许多专用于网络的过滤器。
* `{{ user_password | password_hash('sha512') }}`：生成 sha512 哈希密码。
