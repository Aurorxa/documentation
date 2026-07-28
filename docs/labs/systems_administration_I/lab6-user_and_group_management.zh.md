---
author: Wale Soyinka
contributors: Steven Spencer
tags:
  - lab exercise
  - user management
  - group management
  - systems administration
---

# 实验 6: 用户和组管理

!!! info

    除非特别说明，以下实验指令基于 Rocky Linux 环境。

    输入命令 `lab6-user_and_group_management`，启动一个名为 `lab6-user_and_group_management` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    !!! knowledge "知识要点"

        本实验将带你了解用户和组管理的基础知识：

        * 在本地用户数据库中创建新用户（`useradd`）
        * 管理密码（PAM 和 `/etc/shadow`）
        * 管理用户组
        * 使用 `sudo` 配置管理访问权限

    因为用户管理和安全息息相关，你将在本实验中深入实践如何安全管理用户和组。

## 目标

完成本实验后，你应该能够：

* 使用默认设置创建和删除用户
* 管理不同生命周期阶段的用户密码
* 创建、修改和删除组
* 配置 `sudo` 规则，允许特定用户执行特权命令

## 先决条件

在开始实验之前，你需要：

* 1 台安装了 Rocky Linux 的机器
* 具有 root 访问权限或 sudo 凭据

## 用户管理

在 Rocky Linux 上，添加用户和分配组成员资格是系统管理员的基础工作。你应当了解默认配置，以及 `/etc/login.defs` 和 `/etc/default/useradd` 文件中用户创建的默认值设置。

### 检查和修改默认值

1. 列出 `/etc/login.defs` 文件中特定于用户创建的参数设置，例如：

    - 密码的最大有效天数、最小有效天数和到期警告天数。
    - 创建用户时自动分配的 UID 范围。
    - 是否自动创建用户主目录。
    - 如果创建用户主目录，从哪个骨架目录复制文件。
    - 用户的默认 shell。

    !!! question "问题"

        1. `/etc/login.defs` 中的 `CREATE_HOME` 参数决定什么？它当前的值是什么？
        2. 查看 `/etc/login.defs` 中 `ENCRYPT_METHOD` 的值。它是什么？为什么这很重要？
        3. 描述并解释系统用户 ID (`SYS_UID_MAX`) 和普通用户 ID (`UID_MIN`) 的含义。

2. 列出 `/etc/default/useradd` 文件的内容，并描述它对用户创建的预期效果。

    !!! question "问题"

        1. 什么是骨架目录？它在用户创建过程中被如何使用？
        2. 如何将默认 shell 更改为 `/usr/bin/zsh`？（如果需要，可以先安装 `zsh`。）
        3. 使用 `useradd` 的 `-D` 选项显示当前的默认值。给出命令并显示示例输出。

### 管理用户

1. 创建用户 `sammie`，该用户不需要主目录。设置密码为 `rocky`。验证你是否可以以此用户身份登录。

    !!! question "问题"

        1. 创建没有主目录的用户时，你使用了哪个 `useradd` 选项？
        2. 如果没有主目录，用户从哪里开始他们的 shell 会话？

2. 创建用户 `charlie`，将密码设置为 `rocky`。以该用户身份登录后验证主目录是否存在。

3. 以 root 用户身份运行时，将 `temporary` 设为用户 `charlie` 的密码。

    !!! question "问题"

        `passwd` 命令的这个选项能否让用户下次登录时强制更改密码？为什么或为什么不？

4. 锁定用户 `charlie` 的密码，然后尝试以 `charlie` 身份登录，看看会发生什么。之后，解锁 `charlie` 的密码。

    !!! question "问题"

        密码锁定和密码过期之间的区别是什么？在 `/etc/shadow` 中，什么表示密码锁定或过期？

5. 以 root 用户身份，提示用户 `sammie` 必须在 10 天内更改其密码。切换到 `sammie` 登录后，观察提示，然后为其更改密码。
6. 你还记得如何设置组密码吗？提示：`gpasswd`。

    !!! question "问题"

        1. 在 `/etc/default/useradd` 中禁用自动创建主目录，同时保持 `/etc/login.defs` 中的 `CREATE_HOME yes` 设置，会发生什么？
        2. 主组（primary group）和补充组（supplementary group）的区别是什么？
        3. usermod 命令的 `-aG` 选项有什么作用？省略 `-a` 会有什么风险？

### 给系统添加公告

1. 配置系统每天向通过 SSH 登录的所有用户显示以下公告：

    ```text
    Will the owner of the white Tesla Model 3 please return to your vehicle? Your car is double parked.
    ```

    所有用户都应该看到它。提示：motd。

    !!! question "问题"

        `motd` 代表什么？默认位置在哪里？哪个字段在 `/etc/passwd` 中被检查？

2. 通过运行 `last` 查看最近的系统登录信息，验证谁能看到你的公告。确认前一步的设置是否成功。
3. 检查文件 `/etc/passwd`，识别可能被设置为禁止登录的账户。例如，系统服务账户的 shell 可能被设为 `/sbin/nologin` 或 `/bin/false`。

## 组管理

1. 创建一个名为 `human_resources` 的组，选择对其有意义的 GID。
2. 创建另一个名为 `payroll` 的组，选择对其有意义的 GID。
3. 将用户 `sammie` 添加到 `human_resources` 和 `payroll` 组。
4. 检查 `sammie` 所属的组。使用两种不同的方法。
5. 除了 `sammie` 之外，将 `charlie` 和任何其他你创建的测试用户添加到 `human_resources` 组。
6. 将 `charlie` 添加到 `payroll` 组。
7. 以 `charlie` 身份登录并将 `payroll` 设为其当前会话的主组。验证 `payroll` 是 `charlie` 当前的主要活动组。

    !!! question "问题"

        1. 为什么 `/etc/group` 中的 `x` 占位符很重要？密码哈希实际上存储在哪里？
        2. 将用户添加到组时，你使用了哪个命令？解释该命令中 `-a` 和 `-G` 选项的作用。
        3. 用户创建的文件的默认组所有权由什么决定？如何临时更改活动会话中的默认组？
        4. 彻底删除一个组所需的基本步骤是什么？如果一个"要被删除的组"是某个用户的主组怎么办？

## 管理权限

作为 Linux 管理员，你需要能够委派特权，同时保持系统的安全性。因此，你需要理解并善于使用 `/etc/sudoers` 文件以及 Linux 的权限体系。你还应该了解如何配置和管理 `sudo` 规则。

### 配置 `sudo` 和 `sudoers`

1. 检查用户 `charlie` 或 `sammie` 的当前 sudo 权限。

    !!! question "问题"

        找出 `charlie` 或 `sammie` 可以使用哪些 sudo 权限或命令。命令是什么？是否为空？

2. 尝试以 `charlie` 身份运行 `sudo -l` 看看会发生什么。
3. 配置 Rocky Linux 系统，允许 `charlie` 重启 `sshd` 服务、使用 `dnf` 安装和删除软件包，而无需提供 sudo 密码。

    !!! question "问题"

        在 sudoers 文件中添加用户 `charlie` 的条目，使其无需密码即可执行特定任务。

4. 登出 root，以普通用户 `charlie` 身份登录。尝试安装一个简单的软件包（使用 `dnf`）来测试你的配置。
5. 同时以 `charlie` 的身份重启网络服务。
6. 验证完成后，登出 `charlie` 并切换回 root 以应用进一步的更改。

7. 配置系统，允许 `human_resources` 组的任何成员以 `sammie` 身份执行命令。例如，作为组成员，你应该能够以 `sammie` 身份运行命令。

    !!! question "问题"

        1. `visudo` 是什么工具，为什么建议使用它？
        2. 列出并解释 sudoers 中用户规范和别名行的格式。提供一个通用测试用户条目的示例。
        3. 描述 sudoers 配置文件中 `wheel` 组的默认作用。

## 总结

!!! knowledge "知识要点"

    你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
