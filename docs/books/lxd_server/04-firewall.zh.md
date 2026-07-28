---
title: 4 防火墙设置
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd security
---

# 第 4 章：防火墙设置

在本章中，你需要以 root 身份操作，或者能够通过 `sudo` 成为 root。

与任何服务器一样，你需要确保它对外部世界和局域网都是安全的。你的示例服务器只有一个 LAN（局域网）接口，但完全可以有两个接口，分别面向你的 LAN 和 WAN（广域网）网络。

!!! warning "关于 Rocky Linux 9.x 和 `iptables` 的说明"

    从 Rocky Linux 9.0 开始，`iptables` 及所有相关工具已被正式弃用。这意味着在操作系统的未来版本中，它们将完全消失。本文档的先前版本包含 `iptables` 设置说明，但现已删除。

    对于所有当前版本的 Rocky Linux，建议使用 `firewalld`。

## 防火墙设置 - `firewalld`

对于 _firewalld_ 规则，你需要使用[此基本流程](../../guides/security/firewalld.md)，或熟悉这些概念。我们的假设是：LAN 网络为 192.168.1.0/24，网桥名为 lxdbr0。需要明确的是，你的 LXD 服务器上可能有许多接口，其中一个可能面向 WAN。你还将为桥接网络和本地网络创建一个区域（zone）。这只是为了区域清晰度着想。其他区域名称并不真正适用。此过程假设你已经了解 _firewalld_ 的基础知识。

```bash
firewall-cmd --new-zone=bridge --permanent
```

添加区域后需要重新加载防火墙：

```bash
firewall-cmd --reload
```

你想要允许来自网桥的所有流量。只需添加接口，并将目标从 "default" 改为 "ACCEPT"：

!!! warning

    更改 `firewalld` 区域的目标**必须**使用 `--permanent` 选项，因此不妨也在其他命令中输入此标志，省去 `--runtime-to-permanent` 选项。

!!! Note

    如果你需要创建一个允许对接口或源进行所有访问、但不希望指定任何协议或服务的区域，那么你**必须**将目标从 "default" 更改为 "ACCEPT"。对于你有自定义区域的特定 IP 块，"DROP" 和 "REJECT" 也是如此。需要明确的是，只要你不使用自定义区域，"drop" 区域就会为你处理这个问题。

```bash
firewall-cmd --zone=bridge --add-interface=lxdbr0 --permanent
firewall-cmd --zone=bridge --set-target=ACCEPT --permanent
```

假设没有错误且一切正常工作，只需重新加载：

```bash
firewall-cmd --reload
```

如果现在使用 `firewall-cmd --zone=bridge --list-all` 列出规则，你将看到：

```bash
bridge (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces: lxdbr0
  sources:
  services:
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

请注意，你还希望允许本地接口。同样，自带的区域名称并不适合这种情况。创建一个区域并使用本地接口的源 IP 范围来确保你有访问权限：

```bash
firewall-cmd --new-zone=local --permanent
firewall-cmd --reload
```

添加本地接口的源 IP，并将目标更改为 "ACCEPT"：

```bash
firewall-cmd --zone=local --add-source=127.0.0.1/8 --permanent
firewall-cmd --zone=local --set-target=ACCEPT --permanent
firewall-cmd --reload
```

使用 `firewall-cmd --zone=local --list all` 列出 "local" 区域以确保规则存在，将显示：

```bash
local (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces:
  sources: 127.0.0.1/8
  services:
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

你想要允许来自受信网络的 SSH 访问。这里使用源 IP 和内置的 "trusted" 区域。此区域的目标默认已是 "ACCEPT"。

```bash
firewall-cmd --zone=trusted --add-source=192.168.1.0/24
```

将服务添加到该区域：

```bash
firewall-cmd --zone=trusted --add-service=ssh
```

如果一切正常，将规则移至永久并重新加载：

```bash
firewall-cmd --runtime-to-permanent
firewall-cmd --reload
```

列出 "trusted" 区域将显示：

```bash
trusted (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces:
  sources: 192.168.1.0/24
  services: ssh
  ports:
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

默认情况下，"public" 区域处于启用状态并允许 SSH。出于安全考虑，你不希望 SSH 在 "public" 区域上被允许。确保你的区域设置正确，并且访问服务器的方式是通过某个 LAN IP（如我们的示例）。如果在继续之前不验证这一点，你可能会将自己锁在服务器之外。当你确定从正确的接口具有访问权限后，从 "public" 区域中移除 SSH：

```bash
firewall-cmd --zone=public --remove-service=ssh
```

测试访问并确保你没有锁定自己。如果没有，将规则移至永久，重新加载，并列出 "public" 区域以确保 SSH 已移除：

```bash
firewall-cmd --runtime-to-permanent
firewall-cmd --reload
firewall-cmd --zone=public --list-all
```

你的服务器上可能还有其他需要考虑的接口。你可以在适当的情况下使用内置区域，但如果名称显得不合逻辑，你完全可以添加区域。只需记住，如果你没有需要特别允许或拒绝的服务或协议，则需要更改区域目标。如果可以使用接口（如桥接），则可以这样做。如果你需要对服务进行更细粒度的访问控制，则使用源 IP。
