---
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - lab exercise
  - auditing
  - security
---

# 实验 3: 系统审计

!!! info

    在使用 lab 脚本时，始终使用支持 `tmux` 的环境。

    输入命令 `lab3-auditing_the_system`，启动一个名为 `lab3-auditing_the_system` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，你将学习如何审计你的 Rocky Linux 系统。

    在 Linux 系统上，审计是管理员或安全专业人员用于跟踪和监控系统上安全相关事件的一套规则。审计事件可能包括但不限于：

    * 文件访问
    * 系统调用
    * 运行或终止的命令
    * 用户登录事件

    !!! knowledge "知识要点"
        你已经在之前的实验中接触了一些审计相关内容。例如，你可能已经使用过：
        * `journalctl`：检查登录和 sudo 会话日志
        * `aureport`：通过 `auditd` 查看报告

        在本实验中，你将在安全审计方面更进一步。你将了解并配置审计子系统的核心部分——`auditd`（审计守护进程）。你将学习如何创建和配置审计规则，然后生成一些日志。之后，你将搜索审计日志，生成一些简单的审计报告，并解释你的观察结果。

        !!! note
            auditd 的配置主要集中在一个文件中：`/etc/audit/auditd.conf`。

    让我们开始吧！

## 目标

完成本实验后，你应该能够：

* 安装并启用 auditd
* 配置审计规则和选项
* 使用创建文件、更改权限和读取文件等操作生成审计日志
* 使用 ausearch 和 aureport 查看审计日志和报告

## 先决条件

开始本实验之前，你需要：

* 一台安装了 Rocky Linux 的机器
* 具有 root 权限或 `sudo` 访问权限

## 实验任务

### 检查 auditd 是否已安装

在 Rocky Linux 上，`auditd` 通常已预装并运行。验证这一点：

```bash
rpm -q audit
```

你应该看到如下内容：

```text
audit-3.1.5-7.el9.x86_64
```

或者类似的信息。

### auditd 配置文件的位置

我们之前提到，`auditd` 的核心配置文件是 `/etc/audit/auditd.conf`。这个文件可以修改以配置 auditd 守护进程本身的设置。不过，我们首先关注的是用来控制审计内容的**规则**。

审计**规则**的位置：

| 规则文件 | 说明 |
| --- | --- |
| `/etc/audit/rules.d/audit.rules` | 审计规则的默认、基于发行版的位置 |

请注意，该文件位于 `rules.d/` 目录下，你可以将规则文件放置在该目录中。

### 启用 auditd

1. 确保 auditd 已启用并正在运行：
    ```bash
    systemctl enable auditd && systemctl start auditd
    systemctl status auditd
    ```

2. 确认 auditd 是否处于活动状态并正在运行。

### auditd 规则

为审计而创建的规则会告诉内核的审计子系统要关注哪些事件——例如，哪个用户正在修改哪个文件。

#### 在 auditd 中添加规则

1. 让我们设置一个场景：创建一个文件，监控对它的访问，然后尝试以其他用户身份访问它。首先，在 `/etc/audit/rules.d/` 目录下创建一个名为 `labtest.rules` 的文件，内容如下：

    ```text
    # 监控 /etc/secret_config 的权限
    -w /etc/secret_config -p wa -k secret_config_changes
    # 监控 /home/labuser/important_script.sh 的执行情况
    -w /home/labuser/important_script.sh -p x -k script_execution
    # 记录用户 labuser 的所有命令
    -a always,exit -F arch=b64 -S execve -F uid=1001 -k labuser_commands
    ```

    !!! question "问题"

        列出并解释上述规则文件中各部分的含义。例如，`-w /etc/secret_config -p wa -k secret_config_changes` 这条规则中各部分的含义是什么？尽可能详细地解释。规则中的不同字母标志是什么意思？提示：`-w`、`-p`、`wa` 和 `-k`。

2. 在你喜欢的编辑器中打开当前正在使用的规则文件 `/etc/audit/rules.d/audit.rules`。规则文件中包含大量示例，描述了如何监控特定系统调用、监控特定用户，以及其他相关事项。

    !!! question "问题"

        当 auditd 正在运行时，可以使用两个命令来查看当前活跃的规则。列出它们。提示：查看手册页。

3. 使用 `auditctl` 添加一条临时规则，监控 `/etc/hosts` 文件的更改和访问情况。

    ```bash
    auditctl -w /etc/hosts -p wa -k hosts_changes
    ```

    !!! question "问题"

        你使用了什么命令来列出当前活跃的规则？

4. 重启 auditd 服务以清除临时规则（上文第 3 步中添加的）。重启后，检查已加载的规则——临时规则不应再出现。

    !!! question "问题"

        查找规则文件 audit.rules 中删除已安装包的规则。解释该规则的含义。

5. 从 `audit.rules` 文件中复制任何包含 `-F arch=` 的 5 条规则。将这些规则粘贴到你之前在本实验中创建的 `labtest.rules` 中。

### 使用 auditd 生成报告

你已经创建了规则，记录了一些事件。现在你需要学习如何审计管理员用来搜索事件、生成报告的工具。使用 `ausearch` 和 `aureport` 工具，回答下面列出的问题。

!!! question "问题"

    使用 `ausearch` 和 `aureport` 工具回答以下问题：
    1. 使用 `ausearch` 工具查找特定事件 ID。例如，搜索事件 1234。使用什么命令？
    2. 使用 `ausearch` 查找所有失败的 `sudo` 尝试。你使用什么命令？
    3. 仅搜索使用 `aureport` 生成的摘要报告中有多少登录事件。你使用什么程序或选项？
    4. 生成包含失败认证尝试的摘要报告，并找出有多少次失败，以及尝试了哪些用户。使用什么命令？

## 总结

完成本实验后，你应该对 auditd 的配置和监控有了基础的理解。

!!! knowledge "知识要点"

    你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
