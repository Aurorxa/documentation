---
title: 4 防火墙设置
author: Steven Spencer
contributors: Ezequiel Bruni, Ganna Zhyrnova
tested_with: 9.4
tags:
  - incus 
  - enterprise
  - incus security
---

在本章的全部内容中，你需要是 root 用户或能够通过 `sudo` 成为 root。

与任何服务器一样，你必须确保其免受外部世界和你的 LAN 的威胁。你的示例服务器只有一个 LAN 接口，但有两个接口是可能的，每个接口分别面向你的 LAN 和 WAN 网络。

## 防火墙设置 - `firewalld`

对于 _firewalld_ 规则，你需要使用[此基本步骤](../../guides/security/firewalld.md)或熟悉这些概念。假设 LAN 网络为 192.168.1.0/24，桥接名为 incusbr0。需要明确的是，你的 Incus 服务器上可能有许多接口，其中一个面向你的 WAN。你还需要为桥接网络和本地网络创建一个区域。这仅仅是为了区域的清晰性。其他区域名称不适用。此过程假设你已经了解 _firewalld_ 的基础知识。

```bash
firewall-cmd --new-zone=bridge --permanent
```

添加区域后需要重新加载防火墙：

```bash
firewall-cmd --reload
```

你想要允许来自桥接的所有流量。只需添加接口并将目标从 "default" 更改为 "ACCEPT"：

!!! warning

    更改 `firewalld` 区域的目标*必须*使用 `--permanent` 选项，因此我们最好在其他命令中也输入该标志，并放弃 `--runtime-to-permanent` 选项。

!!! Note

    如果你需要创建一个想要允许对接口或源的所有访问但不希望指定任何协议或服务的区域，那么你*必须*将目标从 "default" 更改为 "ACCEPT"。对于你有自定义区域的特定 IP 块的 "DROP" 和 "REJECT" 也是如此。"drop" 区域将为你处理这个问题，只要你不使用自定义区域。

```bash
firewall-cmd --zone=bridge --add-interface=incusbr0 --permanent
firewall-cmd --zone=bridge --set-target=ACCEPT --permanent
```

假设没有错误且一切仍在工作，进行重新加载：

```bash
firewall-cmd --reload
```

如果现在通过 `firewall-cmd --zone=bridge --list-all` 列出你的规则，你将看到：

```bash
bridge (active)
  target: ACCEPT
  icmp-block-inversion: no
  interfaces: incusbr0
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

请注意，你还想允许你的本地接口。同样，包含的区域名称不适合此用途。创建一个区域并使用本地接口的源 IP 范围来确保你有访问权限：

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

继续使用 `firewall-cmd --zone=local --list all` 列出 "local" 区域以确保你的规则存在，这将显示：

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

你想允许来自你受信任网络的 SSH。为此，允许源 IP 使用内置的 "trusted" 区域。默认情况下，此区域的目标是 "ACCEPT"。

```bash
firewall-cmd --zone=trusted --add-source=192.168.1.0/24
```

将服务添加到该区域：

```bash
firewall-cmd --zone=trusted --add-service=ssh
```

如果一切正常工作，将你的规则移到永久配置并重新加载规则：

```bash
firewall-cmd --runtime-to-permanent
firewall-cmd --reload
```

列出你的 "trusted" 区域将显示：

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

"public" 区域默认启用，并且 SSH 是允许的。为了安全，你不希望在 "public" 区域上允许 SSH。确保你的区域正确，并且你访问服务器的方式是通过某个 LAN IP（在本示例中）。如果在继续之前没有验证这一点，你可能会将自己锁在服务器之外。当你确定有从正确接口的访问权限时，从 "public" 区域移除 SSH：

```bash
firewall-cmd --zone=public --remove-service=ssh
```

测试访问并确保你没有锁定。如果没有，将你的规则移到永久配置，重新加载，并列出 "public" 区域以确保移除 SSH：

```bash
firewall-cmd --runtime-to-permanent
firewall-cmd --reload
firewall-cmd --zone=public --list-all
```

服务器上可能还有其他接口需要考虑。你可以在适当的地方使用内置区域，但如果名称不够好，你可以添加区域。只需记住，如果你没有需要特别允许或拒绝的服务或协议，则需要更改区域目标。如果可以像桥接那样使用接口，也可以这样做。如果你需要对服务进行更细粒度的访问，请改用源 IP。
