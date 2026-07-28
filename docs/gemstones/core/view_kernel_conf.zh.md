---
title: 查看当前内核配置
author: David Hensley
contributors: Steven Spencer
tested_with: 8.5
tags:
  - kernel
  - config
  - modules
  - kmod
---

# 查看当前内核配置

Linux 内核通过两个特殊文件系统存储运行中的内核信息：

- 较旧的 [procfs](https://man7.org/linux/man-pages/man5/procfs.5.html) 挂载 `/proc`（通过 `mount -l -t proc` 验证）
- 较新的 [sysfs](https://man7.org/linux/man-pages/man5/sysfs.5.html) 挂载 `/sys`（通过 `mount -l -t sysfs` 验证）

!!! warning

    检查此处提到的文件时请小心，更改它们可能会改变实际运行中内核的行为！

这两个接口允许您查看和更改当前运行中内核的参数。

需要注意的是，如果您对其中一些文件执行 [`ls -l`](https://man7.org/linux/man-pages/man1/ls.1.html)，它们显示的长度为"0"，但如果您用 [`cat`](https://man7.org/linux/man-pages/man1/cat.1.html) 查看，它们实际上包含数据。大多数是 ASCII 格式且可编辑，但有些是二进制的。无论哪种情况，像 [`file`](https://man7.org/linux/man-pages/man1/file.1.html) 或 [`stat`](https://man7.org/linux/man-pages/man2/lstat.2.html) 这样的命令通常只会返回"empty file"或长度为"0"，但会显示其他信息。

与这些功能交互的首选和标准程序是 [`lsmod`](https://man7.org/linux/man-pages/man8/lsmod.8.html)、[`modinfo`](https://man7.org/linux/man-pages/man8/modinfo.8.html) 和 [`sysctl`](https://man7.org/linux/man-pages/man8/sysctl.8.html) 等。

```bash
sysctl -a | grep -i <keyword>
```

```bash
lsmod | grep -i <keyword>
```

```bash
modinfo <module>
```

使用以下命令查看当前运行中的"kernel release"版本：

`uname -r` 并使用 `$(uname -r)` 在命令中替换其返回值

RHEL 及衍生发行版（Fedora、CentOS Stream、Scientific Linux、RockyLinux、AlmaLinux 等）还会在 Grub2 使用的 `/boot` 目录中将已安装可引导内核的配置存储为 ASCII 文件：

```bash
/boot/config-<kernel-release>
```

要检查当前运行中内核的特定配置值：

```bash
cat /boot/config-$(uname -r) | grep -i <keyword>
```

结果将显示：

- `=m` 如果编译为内核模块
- `=y` 如果静态编译到内核中
- `is not set` 如果该设置被注释掉
- 一个数值
- 一个带引号的字符串值

一些发行版（如 Gentoo 和 Arch）默认使用 `configs` 内核模块来提供 `/proc/config.gz`：

```bash
zcat /proc/config.gz | grep -i <keyword>
zgrep <keyword> /proc/config.gz
```

对于任何发行版，如果您运行的内核同时设置了 `CONFIG_IKCONFIG` 和 `CONFIG_IKCONFIG_PROC`，并且

```bash
ls -lh /sys/module/configs
```

存在且可执行（对于目录意味着可搜索），那么如果 `/proc/config.gz` 不存在，您可以通过以下命令创建它：

```bash
modprobe configs
```

!!! note "启用的仓库"

    本文档目前不涵盖可能来自非默认仓库的内核包，例如：

    appstream-debug、appstream-source、baseos-debug、baseos-source 或 devel

`kernel-devel` 包将用于编译每个已安装标准内核包的配置文件作为 ASCII 文件安装到以下位置：

```bash
/usr/src/kernels/<kernel-release>/.config
```

该文件更常通过 `kernel-core` 包提供的符号链接路径访问：

```bash
/lib/modules/<kernel-release>/build/ -> /usr/src/kernels/<kernel-release>/
```

如果您安装了 `kernel-debug-devel` 包，您还将拥有此目录：

```bash
 /usr/src/kernels/<kernel-release>+debug/
```

您可以在以下任意位置查看用于构建已安装内核的配置值详细信息：

```bash
/lib/modules/<kernel-release>/config
/lib/modules/<kernel-release>/build/.config
/usr/src/kernels/<kernel-release>/.config
/usr/src/kernels/<kernel-release>+debug/.config
```

当前运行内核的已配置模块，无论是编译为内置（即静态编译到内核本身）还是可加载模块，均按其模块名称列在子目录中：

```bash
/sys/module/
```

对于每个已安装的 kernel-release，您可以检查这些文件以查看哪些值被编译到该内核中，以及使用了什么版本的 [GCC](https://man7.org/linux/man-pages/man1/gcc.1.html) 来编译它：

```bash
cat /lib/modules/$(uname -r)/config | grep -i <keyword>
```

```bash
cat /lib/modules/$(uname -r)/build/.config | grep -i <keyword>
```

```bash
cat /usr/src/kernels/$(uname -r)/.config | grep -i <keyword>
```

```bash
cat /usr/src/kernels/$(uname -r)+debug/.config | grep -i <keyword>
```

```bash
ls -lh /sys/module/ | grep -i <keyword>
```

您可以在以下文件中检查内核模块依赖关系：

```bash
/lib/modules/<kernel-release>/modules.dep
```

但是阅读或解析 [`lsmod`](https://man7.org/linux/man-pages/man8/lsmod.8.html) 中"Used-by"字段的输出更为容易。

## 参考

[depmod](https://man7.org/linux/man-pages/man8/depmod.8.html)、[ls](https://man7.org/linux/man-pages/man1/ls.1.html)、[lsmod](https://man7.org/linux/man-pages/man8/lsmod.8.html)、[modinfo](https://man7.org/linux/man-pages/man8/modinfo.8.html)、[modprobe](https://man7.org/linux/man-pages/man8/modprobe.8.html)、[modules.dep](https://man7.org/linux/man-pages/man5/modules.dep.5.html)、[namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html)、[procfs](https://man7.org/linux/man-pages/man5/procfs.5.html)、[sysctl](https://man7.org/linux/man-pages/man8/sysctl.8.html)、[sysfs](https://man7.org/linux/man-pages/man5/sysfs.5.html)、[uname](https://man7.org/linux/man-pages/man8/uname26.8.html)
