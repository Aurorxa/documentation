---
title: NoSleep.sh - 一个简单的配置脚本
author: Andrew Thiesen
tags:
  - configuration
  - server
  - workstation
---

# NoSleep.sh

## 用于编辑 `/etc/systemd/logind.conf` 的 Bash 脚本

此 bash 脚本设计用于编辑 Rocky Linux 服务器或工作站上的 `/etc/systemd/logind.conf` 配置文件。具体来说，它修改 `HandleLidSwitch` 选项并将其设置为 `ignore`。此配置更改通常用于防止在笔记本电脑盖子关闭时系统挂起或执行任何操作。

### 用法

使用脚本的步骤如下：

1. 在 Linux 系统上打开终端。
2. `cd` 到您选择的目录。
3. 通过 `curl` 下载 NoSleep.sh 脚本：`curl -O https://github.com/andrewthiesen/NoSleep.sh/blob/main/NoSleep.sh`
4. 运行 `chmod +x NoSleep.sh` 命令使 NoSleep 脚本可执行。
5. 使用 `sudo ./NoSleep.sh` 命令以 root 身份执行脚本。
6. 脚本将把 `logind.conf` 文件中的 `HandleLidSwitch` 选项更新为 `ignore`。
7. 可选地，系统会提示您重启以使更改立即生效。

### 重要说明

* 此脚本**必须**以 root 或超级用户权限运行才能修改系统文件。
* 假设 `logind.conf` 文件位于 `/etc/systemd/logind.conf`。如果您的系统使用不同的位置，请相应修改脚本。
* 修改系统配置文件可能会带来意外后果。请检查脚本所做的更改，确保它们符合您的需求。
* 建议在执行脚本前采取适当的预防措施，例如备份原始配置文件。
* 重启系统是可选的，但可以确保更改立即生效。执行脚本后系统会提示您重启。

---

请根据您的需要自由定制和使用该脚本。请在系统上运行该脚本之前确保您理解脚本及其影响。
