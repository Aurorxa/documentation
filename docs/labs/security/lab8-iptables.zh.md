---
author: Wale Soyinka
contributors: Steven Spencer
tags:
  - lab exercise
  - iptables
  - security
  - firewall
---

# 实验 8: IPTABLES：Linux 防火墙

!!! info

    输入命令 `lab8-iptables`，启动一个名为 `lab8-iptables` 的 `tmux` 会话。更多信息请参见 [tmux helper scripts](../../../tools_scripts/lab_helpers.md)。

    在本实验中，你将在 Rocky Linux 上锻炼使用 `iptables` 的技能。

    作为 Linux 管理员，配置系统的防火墙是一项关键的安全技能。Linux 内核内置了一个名为 `netfilter` 的数据包过滤框架，而 `iptables` 系统使用 netfilter 框架来创建路由器、网络地址转换（NAT）以及包过滤规则。最新版本的 Linux 内核也已集成了 `nftables`（`iptables` 的继任者）。很多新发行版默认使用 `nftables`。在本实验指南中，你将学习如何使用一些最常用的基于 `iptables` 的防火墙规则。

    !!! knowledge "知识要点"

        本实验将允许你使用 `iptables` 命令行工具。你将亲手演练一些最常用的 `iptables` 规则，以理解和创建 Linux 防火墙。

        `nftables` 是 Linux 内核数据包过滤框架的现代 Linux 版本，它取代了旧的 `iptables`（以及 `ip6tables`、`arptables` 等）基础设施。

        我们将继续强调 `iptables`，因为在可预见的未来，它可能会继续存在并被使用。

## 目标

完成本实验后，你应该能够：
* 检查当前的 iptables 配置
* 熟悉默认的 iptables 链
* 理解常见的 iptables 规则

## 先决条件

在开始实验之前，你需要：
* 1 台安装了 Rocky Linux 的机器（1 个节点）。
* 具有 root 访问权限或 sudo 凭据。
* 在你的机器上禁用或关闭 `firewalld`。

## 介绍

在你的 Rocky Linux 系统上，输入以下命令查看当前的 `iptables` 配置：

```bash
iptables -L
```

输出类似于：

```bash
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

该输出示例显示了数据包的处理链（chain），以及每个链的关联**规则**。`iptables` 命令允许你控制 Linux 防火墙如何处理传入、传出的流量，以及是否转发数据包。默认情况下，有三个链：

- `INPUT` — 应用于传入本地套接字的数据包
- `FORWARD` — 应用于通过系统路由的数据包
- `OUTPUT` — 应用于本地生成的数据包

每个链都有一个**策略**（policy），它决定了当数据包不属于任何特定规则时的默认处理方式。默认设置是 `ACCEPT`，这意味着每个链都会接受所有数据包，除非有一个 `REJECT` 或 `DROP` 规则覆盖了这个默认行为。设置多个规则后，防火墙的处理逻辑是：数据包逐一经过每个规则，使用遇到的第一个适用规则。如果没有规则匹配，则应用链的策略（默认 `ACCEPT`）。一旦有规则匹配，防火墙就会停止继续遍历链中的剩余规则，该链对该数据包的处理完成。

!!! question "问题"

    1. 你能解释一下上面的 `ACCEPT` 规则吗？例如，INPUT 链中的第二条规则做什么？
    2. 列出并解释给定链的处理策略。
    3. 使用什么命令可以列出给定链的详细过滤和包/字节计数信息？

在开始之前，你可能需要确保系统将你当前活动的 SSH 会话添加到白名单中，这样之后就不会被防火墙规则意外踢下线。运行以下命令可以让你放心：

```bash
iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp -m tcp --dport 22 -j ACCEPT
```

## 练习

!!! note "注意！"

    建议在开始更改防火墙规则之前先保存一份副本。如果你的实验室支持快照，可以在开始之前拍个快照。如果不行，可以通过运行以下命令保存当前 iptables 规则的备份：

    ```bash
    iptables-save > ~/iptables.backup
    ```

1. **拒绝 ICMP 协议**

    你的防火墙已启用，并且你希望控制谁可以 ping 你的机器以及你的机器可以 ping 谁。

    * 在 chains 中添加一条拒绝从任何人那里接收 `icmp` 数据包的规则。
    * 在 chains 中添加一条拒绝向任何人发送 `icmp` 数据包的规则。
    * 验证你的工作：确保你的机器不再接受 `ICMP`。你可以通过尝试 ping 你的机器并观察结果来验证。
    * 正确执行此练习后，恢复 ICMP 协议访问。

    !!! question "问题"

        1. 列出你用于此练习的命令。
        2. 如何从 INPUT 链中删除规则——例如，删除你为阻止 ping 而添加的规则的第 1 个实例？

2. **根据源或目标地址过滤流量**

    为了展示 `iptables` 的威力以及你可以控制的消息粒度和类型，我们将向你展示你可以允许或拒绝某些类型的连接，不仅基于使用的协议类型，还可以基于源地址、MAC 地址、目标端口等。

    * 从你的系统 ping 众所周知的 Google DNS 服务器，即 `8.8.8.8`。它成功了吗？
    * 现在创建一个防火墙规则来阻止 `8.8.8.8`。
    * 再次从你的系统 ping `8.8.8.8`。这次它成功了吗？你应该看到目标不可达的错误。
    * 验证 INPUT 链规则已正确创建并有效：

        ```bash
        iptables -L
        ```

    * 删除规则以允许来自任何地址的 ping 请求。
    * 恢复前一步中删除的规则，这次通过使用 `iptables-restore`。

    !!! question "问题"

        1. 对于基于地址的过滤，规则是否需要放在特定的链中？
        2. 提供一个命令示例，以允许来自网络 10.0.1.0/24 的 ping 请求，同时拒绝其他所有 ping 请求。
        3. `iptables save` 和 `iptables restore` 命令的作用是什么？
        4. 当遇到 `request timed out` 与其他错误消息时，背后可能代表什么含义？
        5. 描述插入规则 `iptables -I INPUT 2 -s 192.168.1.100 -j DROP` 的效果。将此命令与在末尾追加规则进行对比。

3. **保护特定端口的安全**

    管理员可以严格控制系统上哪些端口可以接收或发起流量。

    * 在你的系统上安装 Web 服务器（如 Apache 或 Nginx）或其他常见服务。提示：使用你将使用的任何 HTTP 服务器的默认端口（如 TCP 80）。
    * 添加一条防火墙规则，只允许你的本地网络 `10.0.2.0/24` 访问 HTTP 服务。
    * 配置完成后，从其他系统访问你的 HTTP 服务，验证你的设置。

    !!! question "问题"

        如果收到连接被拒绝的消息，可能的原因是什么？对可能的解决方案你有哪些看法？

4. **使用其他 iptables 目标（`target`）**

    通常，iptables 规则的目标（即 `-j` 后面的目标）包括 `ACCEPT`、`DROP` 和 `REJECT`。但还有其他可用的选项，如 `LOG`。

    研究 `LOG` 目标，并练习其用法。

    !!! question "问题"

        1. 简要讨论使用 `LOG` 目标的价值。
        2. 在规则中同时使用 `DROP` 和 `LOG` 目标选项的语法是什么？

5. **探索 `nftables`**

    `nftables` 在较新的 Rocky Linux 版本上默认可用。

    研究将现有 `iptables` 规则迁移到 `nftables` 语法和文件。

    !!! question "问题"

        有哪些工具可以帮助你迁移 `iptables` 规则到 `nftables`？请提供示例用法。

## 总结

最后，我们确保你的系统在重新启动后能够保持规则——毕竟，你辛苦创建的规则不应该在重启后丢失。根据你的结果，完成以下步骤：

* 使用适当的方法保存当前规则（例如 `iptables-save`）。
* 确保当前规则在系统重启后保留。

!!! knowledge "知识要点"

    你已经掌握了 Rocky Linux——在 Internet 上或许充满挑战，但在你的指导下，你已准备好迎接一切！
