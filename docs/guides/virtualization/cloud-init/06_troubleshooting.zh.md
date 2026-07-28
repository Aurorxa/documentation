---
title: 6. cloud-init 故障排除
author: Wale Soyinka
contributors: Steven Spencer, Ganna Zhyrnova
tags:
  - cloud-init
  - rocky linux
  - cloud
  - automation
  - troubleshooting
---

## cloud-init 故障排除

在任何复杂的自动化系统中，事情最终都会出错。当 `cloud-init` 配置失败时，知道如何系统地诊断问题是一项关键技能。本章是 `cloud-init` 取证 (forensics) 指南，涵盖客机端 (in-guest) 和宿主机端 (on-host) 的故障排除技术。

## 1. 客机端故障排除工具包

当你能访问一个运行中的实例时，`cloud-init` 提供了几个命令和日志来显示发生了什么。

### 支柱 1：状态命令 (`cloud-init status`)

这是你的第一站。它提供了 `cloud-init` 状态的高级摘要。

* **检查 `cloud-init` 是否完成：** `cloud-init status`
    （成功运行将显示 `status: done`）

* **等待 `cloud-init` 完成：** `cloud-init status --wait`
    （这在脚本中很有用，可以暂停执行直到 `cloud-init` 完成）

### 支柱 2：主日志 (`/var/log/cloud-init.log`)

这个文件是真相的黄金来源：每个阶段和模块的详细、按时间顺序的记录。当你想知道 *确切* 发生了什么时，查看这里。在该文件中搜索 `ERROR` 或 `WARNING` 通常会带你直接找到问题。

### 支柱 3：输出日志 (`/var/log/cloud-init-output.log`)

此日志捕获由 `cloud-init` 执行的所有脚本的完整 `stdout` (标准输出) 和 `stderr` (标准错误)（例如来自 `runcmd` 的脚本）。如果一个模块运行了但其中的脚本失败了，错误消息将在此文件中。

**动手实践：调试失败的 `runcmd`**

1. 创建一个 `runcmd` 存在微妙错误的 `user-data.yml`：

    ```bash
    cat <<EOF > user-data.yml
    #cloud-config
    runcmd:
      - [ ls, /non-existent-dir ]
    EOF
    ```

2. 使用此数据启动一个虚拟机。`cloud-init status` 将报告 `status: done`，因为 `runcmd` 模块本身执行成功了。

3. 然而，`/var/log/cloud-init-output.log` 将包含来自 `ls` 命令的实际错误，显示失败的内容：

    ```
    ls: cannot access '/non-existent-dir': No such file or directory
    ```

## 2. 使用 `libguestfs-tools` 进行宿主机端故障排除

有时，虚拟机可能完全无法启动，使得客机端工具失效。在这些情况下，你可以通过使用强大的 `libguestfs-tools` 工具套件直接从宿主机检查虚拟机的磁盘镜像来诊断问题（使用 `sudo dnf install libguestfs-tools` 安装）。

### `virt-cat`：从客机磁盘读取文件

`virt-cat` 允许在不挂载的情况下从虚拟机磁盘镜像读取文件。这对于从不启动的实例中获取日志文件非常理想。

```bash
# 从宿主机读取虚拟机磁盘的 cloud-init.log
sudo virt-cat -a /path/to/your-vm-disk.qcow2 /var/log/cloud-init.log
```

### `virt-inspector`：深度系统检查

`virt-inspector` 生成虚拟机的操作系统、应用程序和配置的详细 XML 报告。这对于自动化分析非常强大。

* **获取完整报告：**

    ```bash
    sudo virt-inspector -a your-vm-disk.qcow2 > report.xml
    ```

* **执行定向查询：** 你可以将 XML 管道输出到 `xmllint` 以提取特定信息。此示例检查镜像中安装的 `cloud-init` 版本：

    ```bash
    sudo virt-inspector -a your-vm-disk.qcow2 | xmllint --xpath "//application[name='cloud-init']/version/text()" -
    ```

## 3. 常见陷阱与如何避免

### 陷阱 1：YAML 和 Schema (模式) 错误

无效的 YAML 是最常见的失败来源。一个更高级的问题是语法上有效的 YAML 文件违反了 `cloud-init` 的预期结构（例如模块名称的拼写错误）。

* **解决方案：** 在启动之前使用 `cloud-init schema` 命令来验证你的配置。它会捕获 YAML 错误和结构错误。

    ```bash
    # 根据官方 schema 验证你的 user-data 文件
    cloud-init schema --config-file user-data.yml
    ```

    如果文件有效，它将打印 `Valid cloud-config: user-data.yml`。如果无效，它会提供详细的错误信息。

### 陷阱 2：依赖网络的模块失败

如果网络无法启动，诸如 `packages` 之类的模块将会失败。检查你的网络配置以及 `/var/log/cloud-init.log` 中的 `Network` 阶段。

## 4. 控制 `cloud-init` 的执行

* **强制重新运行：** 要在运行的虚拟机上测试更改，运行 `sudo cloud-init clean --logs` 然后 `sudo reboot`。
* **禁用 `cloud-init`：** 要防止 `cloud-init` 在后续启动时运行，创建一个标志文件：`sudo touch /etc/cloud/cloud-init.disabled`。
* **每次启动都运行 (`bootcmd`)：** 对于必须在每次启动时运行的脚本，使用 `bootcmd` 模块。这很少见，但对特定的诊断很有用。

## 下一步

你现在已经配备了用于客机端和宿主机端故障排除的强大工具集。在最后一章中，我们将审视 `cloud-init` 项目本身，为你探索其源代码并向社区做出贡献做好准备。
