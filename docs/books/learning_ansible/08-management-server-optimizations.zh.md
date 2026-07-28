---
title: 管理服务器优化
author: Antoine Le Morvan
contributors: Steven Spencer, Ganna Zhyrnova
update: 06-Dec-2021
---

# 管理服务器优化

在本章中，我们将回顾可能有助于优化 Ansible 管理服务器的配置选项。

## `ansible.cfg` 配置文件

一些有趣的配置选项需要说明：

* `forks`：默认值为 5，这是 Ansible 并行启动以与远程主机通信的进程数。此数字越高，Ansible 能够同时管理的客户端就越多，从而加快处理速度。你可以设置的值取决于管理服务器的 CPU/RAM 限制。请注意，默认值 `5` 非常小，Ansible 文档指出许多用户将其设置为 50，甚至 500 或更多。

* `gathering`：此变量更改收集事实（facts）的策略。默认情况下，值为 `implicit`，这意味着将系统地收集事实。将此变量切换为 `smart` 允许仅在事实尚未被收集时收集。与事实缓存（见下文）配合使用，此选项可以极大提高性能。

* `host_key_checking`：注意你的服务器安全性！但是，如果你控制你的环境，禁用远程服务器的密钥检查并在连接时节省一些时间可能是有趣的。你也可以在远程服务器上禁用 SSH 服务器的 DNS 使用（在 `/etc/ssh/sshd_config` 中，选项 `UseDNS no`），此选项在连接时浪费时间，并且大多数时候仅在连接日志中使用。

* `ansible_managed`：此变量默认包含 `Ansible managed`，通常用于部署在远程服务器上的文件模板中。它允许你向管理员指定该文件是自动管理的，他们对文件所做的任何更改都可能会丢失。让管理员获得更完整的消息可能很有意义。但要注意，如果你更改此变量，可能会导致守护进程重新启动（通过与模板关联的 handlers）。

* `ssh_args = -C -o ControlMaster=auto -o ControlPersist=300s -o PreferredAuthentications=publickey`：指定 ssh 连接选项。通过禁用除公钥之外的所有认证方法，你可以节省大量时间。你也可以增加 `ControlPersist` 以提高性能（文档建议相当于 30 分钟的值可能是合适的）。与客户端的连接将保持打开更长时间，并且在重新连接到同一服务器时可以重复使用，这是一个显著的时间节省。

* `control_path_dir`：指定连接套接字的路径。如果此路径太长，可能会导致问题。考虑将其更改为简短的内容，如 `/tmp/.cp`。

* `pipelining`：将此值设置为 `True` 可通过减少运行远程模块时所需的 SSH 连接数来提高性能。你必须首先确保 `sudoers` 选项中禁用了 `requiretty` 选项（参见文档）。

## 缓存事实（facts）

收集事实是一个可能需要一些时间的过程。禁用不需要它的 playbook 的事实收集（通过 `gather_facts` 选项）或将这些事实在缓存中保留一定时间（例如 24 小时）可能会有趣。

这些事实可以轻松存储在 `redis` 数据库中：

```bash
sudo yum install redis
sudo systemctl start redis
sudo systemctl enable redis
sudo pip3 install redis
```

不要忘记修改 ansible 配置：

```bash
fact_caching = redis
fact_caching_timeout = 86400
fact_caching_connection = localhost:6379:0
```

要检查正确操作，只需查询 `redis` 服务器：

```bash
redis-cli
127.0.0.1:6379> keys *
127.0.0.1:6379> get ansible_facts_SERVERNAME
```

## 使用 Vault

各种密码和秘密不能以明文形式与 Ansible 源代码一起存储，无论是在管理服务器本地还是在可能的源代码管理器上。

Ansible 建议使用加密管理器：`ansible-vault`。

原理是使用 `ansible-vault` 命令加密一个变量或整个文件。

Ansible 将能够在运行时通过从文件（例如）`/etc/ansible/ansible.cfg` 中检索加密密钥来解密此文件。后者也可以是 python 脚本或其他。

编辑 `/etc/ansible/ansible.cfg` 文件：

```bash
#vault_password_file = /path/to/vault_password_file
vault_password_file = /etc/ansible/vault_pass
```

将密码存储在此文件 `/etc/ansible/vault_pass` 中，并分配必要的限制性权限：

```bash
mysecretpassword
```

然后你可以使用以下命令加密你的文件：

```bash
ansible-vault encrypt myfile.yml
```

通过 `ansible-vault` 加密的文件可以通过其头部轻松识别：

```text
$ANSIBLE_VAULT;1.1;AES256
35376532343663353330613133663834626136316234323964333735363333396136613266383966
6664322261633261356566383438393738386165333966660a343032663233343762633936313630
34373230124561663766306134656235386233323964336239336661653433663036633334366661
6434656630306261650a313364636261393931313739363931336664386536333766326264633330
6334
```

一旦文件被加密，仍然可以使用以下命令编辑它：

```bash
ansible-vault edit myfile.yml
```

你也可以将密码存储转移到任何密码管理器。

例如，要检索存储在 rundeck vault 中的密码：

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import urllib.request
import io
import ssl

def get_password():
    '''
    :return: Vault password
    :return_type: str
    '''
    ctx = ssl.create_default_context()
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE

    url = 'https://rundeck.rockylinux.org/api/11/storage/keys/ansible/vault'
    req = urllib.request.Request(url, headers={
                          'Accept': '*/*',
                          'X-Rundeck-Auth-Token': '****token-rundeck****'
                          })
    response = urllib.request.urlopen(req, context=ctx)

    return response.read().decode('utf-8')

if __name__ == '__main__':
    print(get_password())
```

## 使用 Windows 服务器

需要在管理服务器上安装几个软件包：

* 通过包管理器：

```bash
sudo dnf install python38-devel krb5-devel krb5-libs krb5-workstation
```

并配置 `/etc/krb5.conf` 文件以指定正确的 `realms`：

```bash
[realms]
ROCKYLINUX.ORG = {
    kdc = dc1.rockylinux.org
    kdc = dc2.rockylinux.org
}
[domain_realm]
  .rockylinux.org = ROCKYLINUX.ORG
```

* 通过 python 包管理器：

```bash
pip3 install pywinrm
pip3 install pywinrm[credssp]
pip3 install kerberos requests-kerberos
```

## 使用 IP 模块

网络模块通常需要 `netaddr` python 模块：

```bash
sudo pip3 install netaddr
```

## 生成 CMDB

一个工具 `ansible-cmdb` 已被开发出来，用于从 ansible 生成 CMDB（配置管理数据库）。

```bash
pip3 install ansible-cmdb
```

事实必须由 ansible 使用以下命令导出：

```bash
ansible --become --become-user=root -o -m setup --tree /var/www/ansible/cmdb/out/
```

然后你可以生成一个全局 `json` 文件：

```bash
ansible-cmdb -t json /var/www/ansible/cmdb/out/linux > /var/www/ansible/cmdb/cmdb-linux.json
```

如果你更喜欢 web 界面：

```bash
ansible-cmdb -t html_fancy_split /var/www/ansible/cmdb/out/
```
