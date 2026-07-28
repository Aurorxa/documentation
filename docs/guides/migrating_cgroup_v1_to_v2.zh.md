---
title: 在 Rocky Linux 上从 cgroups v1 迁移到 v2
author: Howard Van Der Wal
ai_contributors: Claude (claude-opus-4-6)
tested with: 8.10
tags:
- cgroups
- containers
- hpc
- podman
- slurm
---

## AI 使用说明

本文档遵循[此处提供的 AI 贡献政策](contribute/ai-contribution-policy.md)。如果您发现操作说明中有任何错误，请告知我们。

## 简介

控制组 (cgroups) 是 Linux 内核的一项功能，它将进程组织为分层组，以管理和限制 CPU、内存和 I/O 等系统资源。cgroup v2 接口^1^（也称为统一层级）用单一、一致的树结构取代了原有的 cgroup v1 接口。

Rocky Linux 8 默认使用 cgroup v1。在以下情况下需要迁移到 cgroup v2：

- 运行无根 Podman 容器（cgroup v1 不支持无根 cgroups）
- 在较新硬件上使用 Slurm 25.x 或更高版本，cgroup v1 会导致不稳定性
- 为迁移到 Rocky Linux 9 及更高版本做准备，其中 cgroup v2 是默认设置

本指南介绍了在 Rocky Linux 8.10 上启用 cgroup v2、验证迁移、解决 systemd 239 的常见问题、配置无根 Podman，以及与 HPC 计算节点上的 Slurm 集成。

## 前置条件

在开始之前，请确保您具有：

- 一台运行 Rocky Linux 8.10 的系统
- Root 或 sudo 访问权限
- 系统的当前备份或快照
- 了解当前使用的 cgroup 版本

验证当前的 cgroup 版本：

```bash
mount | grep cgroup
```

如果输出显示 `tmpfs on /sys/fs/cgroup type tmpfs` 及其下方的多个 `cgroup` 挂载点，则表示您正在运行 cgroup v1。单个 `cgroup2` 挂载点表示 cgroup v2 已启用。

检查内核和 systemd 版本：

```bash
uname -r
systemctl --version
```

Rocky Linux 8.10 搭载 systemd 239，该版本对 cgroup v2 存在已知限制^5^。本指南解决了这些限制。

## 启用 cgroup v2

通过添加内核引导参数启用统一 cgroup 层级：

```bash
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
```

!!! warning "Slurm 环境的额外参数"

    如果您正在运行 Slurm，SchedMD 建议^2^添加两个额外参数，以防止 cgroup v1 控制器与 cgroup v2 一起加载：

    ```bash
    sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1 systemd.legacy_systemd_cgroup_controller=0 cgroup_no_v1=all"
    ```

    `systemd.legacy_systemd_cgroup_controller=0` 参数防止 systemd 挂载混合的 cgroup v1/v2 层级。`cgroup_no_v1=all` 参数在内核级别禁用所有 cgroup v1 控制器。

重启系统以使更改生效：

```bash
sudo reboot
```

## 验证迁移

重启后，确认 cgroup v2 已激活。

检查 cgroup 挂载：

```bash
mount -l | grep cgroup
```

预期输出：

```text
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate)
```

验证可用控制器：

```bash
cat /sys/fs/cgroup/cgroup.controllers
```

预期输出：

```text
cpuset cpu io memory hugetlb pids rdma
```

检查进程 cgroup 分配：

```bash
cat /proc/self/cgroup
```

预期输出（单行，统一层级）：

```text
0::/user.slice/user-1000.slice/session-1.scope
```

如果看到带有编号控制器的多行（例如 `1:memory:/`、`2:cpu:/`），则 cgroup v1 仍处于活动状态。验证 grubby 参数是否已正确应用：

```bash
cat /proc/cmdline | grep unified
```

## 在 systemd 239 上启用缺失的控制器

Rocky Linux 8.10 搭载 systemd 239，该版本默认不启用所有 cgroup v2 控制器。迁移后，您可能会发现子目录控制中缺少 `cpu` 和 `cpuset` 控制器：

```bash
cat /sys/fs/cgroup/cgroup.subtree_control
```

如果输出仅显示 `memory pids` 而非 `cpuset cpu memory pids`，请按照以下步骤操作。

!!! warning "systemd 239 cpuset 限制"

    早于 244 的 systemd 版本不原生支持 cgroup v2 中的 cpuset 接口。Rocky Linux 8.10 搭载 systemd 239。启用 CPU 计费会添加 `cpu` 控制器，而 `cpuset` 控制器必须单独启用。需要重启以使更改完全生效。

### 步骤 1：启用 CPU 计费

编辑 `/etc/systemd/system.conf` 并将 `DefaultCPUAccounting=yes` 设置为：

```bash
sudo sed -i 's/^#DefaultCPUAccounting=no/DefaultCPUAccounting=yes/' /etc/systemd/system.conf
```

如果该行不存在，请在 `[Manager]` 部分下添加：

```ini
[Manager]
DefaultCPUAccounting=yes
```

### 步骤 2：重新加载并重启

```bash
sudo systemctl daemon-reload
sudo reboot
```

### 步骤 3：启用 cpuset 控制器

重启后，验证当前状态：

```bash
cat /sys/fs/cgroup/cgroup.subtree_control
```

您应该看到 `cpu memory pids`。`cpuset` 控制器可用但尚未在子树中。启用它：

```bash
echo "+cpuset" | sudo tee /sys/fs/cgroup/cgroup.subtree_control
```

验证所有控制器现已激活：

```bash
cat /sys/fs/cgroup/cgroup.subtree_control
```

预期输出：

```text
cpuset cpu memory pids
```

!!! note "持久化 cpuset 启用"

    手动 `echo "+cpuset"` 命令在重启后不会保留。在 Slurm 计算节点上，`/etc/slurm/cgroup.conf` 中的 `EnableControllers=yes` 设置会自动处理控制器委托，包括 cpuset，因此不需要手动启用。对于非 Slurm 系统，将该命令添加到 systemd 服务中以使其持久化。在某些系统上，如果实时（FIFO/RR）内核进程正在运行，`echo "+cpuset"` 可能会失败并显示 `Invalid argument`。在这种情况下，设置 `DefaultCPUAccounting=yes` 后重启可能解决问题。

## PAM 配置

通过 ssh 连接时，`pam_systemd.so` 模块将用户会话注册到 systemd，这会创建 `/run/user/$UID` 目录并设置 `XDG_RUNTIME_DIR` 和 `DBUS_SESSION_BUS_ADDRESS` 环境变量。这些是无根容器和其他通过 D-Bus 通信的用户级服务所必需的。

在默认的 Rocky Linux 8.10 安装中，`pam_systemd.so` 通过 `/etc/pam.d/sshd` 中的 `password-auth` 包含项加载。检查它是否处于活动状态：

```bash
grep pam_systemd /etc/pam.d/sshd /etc/pam.d/password-auth
```

如果在 `/etc/pam.d/password-auth` 中该行被注释掉（在有意禁用它的 HPC 计算节点上很常见），可以通过在 `pam_selinux.so open env_params` 行之后直接添加到 `/etc/pam.d/sshd` 来重新启用：

```bash
sudo sed -i '/pam_selinux.so open env_params/a session    optional     pam_systemd.so' /etc/pam.d/sshd
```

重启 `systemd-logind` 以使更改生效：

```bash
sudo systemctl restart systemd-logind
```

通过打开一个新的 ssh 会话（不要使用 `sudo -i -u` 或 `su`，因为它们不会触发 PAM 会话模块）并检查以下内容来验证：

```bash
echo $XDG_RUNTIME_DIR
echo $DBUS_SESSION_BUS_ADDRESS
cat /proc/self/cgroup
```

预期输出：

```text
/run/user/<UID>
unix:path=/run/user/<UID>/bus
0::/user.slice/user-<UID>.slice/session-<N>.scope
```

!!! warning "使用 pam_slurm_adopt 的 HPC 计算节点"

    如果您在计算节点上使用 `pam_slurm_adopt`^6^，请勿在 PAM 配置中启用 `pam_systemd.so`。`pam_slurm_adopt` 文档声明它应为最后一个会话模块，且 `pam_systemd.so` 应被禁用。请参阅下方的"HPC 计算节点配置"部分了解推荐方法。

## 无根 Podman 兼容性

无根 Podman^7^ 需要 cgroup v2。使用 cgroup v1 时，内核不支持无根 cgroup 管理。

### Cgroup 管理器配置

迁移到 cgroup v2 后，Podman 默认使用 `systemd` 作为其 cgroup 管理器。这需要一个带有 D-Bus 套接字的活跃 systemd 用户会话。在用户会话不可用的环境中（如 HPC 计算节点或批处理作业），请配置 Podman 改用 `cgroupfs`。

创建或编辑 `~/.config/containers/containers.conf`：

```ini
[engine]
cgroup_manager = "cgroupfs"
```

对于系统级配置，编辑 `/etc/containers/containers.conf`。

### 已知的 podman stats 警告

在配备 Podman 4.9.x 的 Rocky Linux 8.10 上，运行 `podman stats` 可能会产生警告：

```text
WARN[0005] Failed to retrieve cgroup stats: open /sys/fs/cgroup/.../memory.stat: no such file or directory
WARN[0005] Failed to retrieve cgroup stats: open /sys/fs/cgroup/.../pids.current: no such file or directory
```

!!! note "Podman 4.9.x 中的表面性警告"

    这些警告是由于容器 cgroup 目录在 `podman stats` 完成读取之前被移除的竞争条件引起的。这仅是表面性警告，不影响容器功能。修复^4^已合并到 Podman 5.3 中，但未被回溯到 Rocky Linux 8 附带的 4.9.x 系列。

!!! warning "NFS 支持的容器存储"

    如果您的容器存储路径在 NFS 上，Podman 可能会显示：

    ```text
    WARN[0000] Network file system detected as backing store. Enforcing overlay option `force_mask="700"`. Add it to storage.conf to silence this warning
    ```

    NFS 不支持无根 Podman 所需的 `chown` 操作。对于容器图根使用本地存储（如 XFS），对于运行根使用 tmpfs（`/dev/shm`）。示例 `~/.config/containers/storage.conf`：

    ```ini
    [storage]
    driver = "overlay"
    graphroot = "/local/podman/graphroot"
    runroot = "/dev/shm/runroot"

    [storage.options]
    mount_program = "/usr/bin/fuse-overlayfs"
    ```

## Slurm 注意事项

Slurm 支持 cgroup v2 进行作业资源隔离^2^。以下配置更改适用于 Slurm 控制器和计算节点。

### Slurm cgroup 配置

在所有节点上编辑 `/etc/slurm/cgroup.conf`：

```ini
ConstrainCores=yes
ConstrainRAMSpace=yes
ConstrainSwapSpace=yes
ConstrainDevices=no
AllowedSwapSpace=0
EnableControllers=yes
```

`EnableControllers=yes` 设置告诉 Slurm 在 cgroup v2 子树中启用它需要的 cgroup 控制器^3^。

### Slurmd 服务委托

确保 `slurmd` 服务已启用 cgroup 委托^5^。创建 systemd 下挂配置：

```bash
sudo mkdir -p /etc/systemd/system/slurmd.service.d
cat <<'EOF' | sudo tee /etc/systemd/system/slurmd.service.d/delegate.conf
[Service]
Delegate=yes
EOF
sudo systemctl daemon-reload
sudo systemctl restart slurmd
```

!!! tip "混合 cgroup v1 和 v2 集群"

    如果您的集群中有些节点使用 cgroup v1 而其他节点使用 cgroup v2，请在 `slurm.conf` 中使用默认的 `CgroupPlugin=autodetect` 设置。每个节点将独立检测自己的 cgroup 版本。除非所有节点已完成迁移，否则不要设置 `CgroupPlugin=cgroup/v2`。

### 重启 Slurm 控制器

更改 `cgroup.conf` 后，在控制器节点上重启 `slurmctld`：

```bash
sudo systemctl restart slurmctld
```

验证配置：

```bash
scontrol show config | grep EnableControllers
scontrol show config | grep CgroupPlugin
```

## HPC 计算节点配置（cgroupfs 方法）

在由 Slurm 管理并使用 `pam_slurm_adopt` 的 HPC 计算节点上，推荐的方法是使用 `cgroupfs` 作为 Podman 的 cgroup 管理器，而不是依赖 systemd 用户会话。这消除了在共享计算节点上对 `pam_systemd.so` 和 linger 模式的需求。

此方法通过与 SchedMD 的联合通话得到验证，他们确认 Podman 可以通过使用 `cgroupfs` 作为 cgroup 管理器，在没有 systemd 的 linger 模式或 `XDG_RUNTIME_DIR` 的情况下运行。

### 容器配置

在计算节点上创建 `/etc/containers/containers.conf`：

```ini
[containers]
cgroupns = "host"

[engine]
cgroup_manager = "cgroupfs"
runtime = "crun"

[engine.runtimes]
crun = ["/usr/bin/crun"]
```

`cgroupns = "host"` 设置将容器保留在 Slurm 作业 cgroup 层级内，确保 Slurm 资源限制（CPU、内存）对容器生效。

### 计算节点的 PAM 配置

在使用 `pam_slurm_adopt` 的计算节点上，在所有 PAM 文件中保持 `pam_systemd.so` 禁用：

- `/etc/pam.d/system-auth`：`#-session optional pam_systemd.so`
- `/etc/pam.d/password-auth`：`#-session optional pam_systemd.so`
- `/etc/pam.d/sshd`：`#session optional pam_systemd.so`

### Slurm prolog 和 epilog 脚本

由于 `pam_systemd.so` 被禁用，Slurm prolog 和 epilog 脚本为每个作业创建和清理 `/run/user/$UID`。

创建 `/etc/slurm/prolog.sh`：

```bash
#!/bin/bash
RUN_DIR="/run/user/$(id -u "$SLURM_JOB_USER")"
if [ ! -d "$RUN_DIR" ]; then
    mkdir -p "$RUN_DIR"
    chown "$SLURM_JOB_USER":"$(id -gn "$SLURM_JOB_USER")" "$RUN_DIR"
    chmod 700 "$RUN_DIR"
fi
exit 0
```

创建 `/etc/slurm/epilog.sh`：

```bash
#!/bin/bash
RUN_DIR="/run/user/$(id -u "$SLURM_JOB_USER")"
OTHER_JOBS=0
for pid in $(scontrol listpids 2>/dev/null | awk -v jid="$SLURM_JOB_ID" 'NR!=1 { if ($2 != jid && $1 != "-1"){print $1} }'); do
    ps --noheader -o euser p "$pid" 2>/dev/null | grep -q "$SLURM_JOB_USER" && OTHER_JOBS=1
done
if [ "$OTHER_JOBS" -eq 0 ]; then
    [ -d "$RUN_DIR" ] && rm -rf "$RUN_DIR"
fi
exit 0
```

创建 `/etc/slurm/task_prolog.sh`：

```bash
#!/bin/bash
echo "export XDG_RUNTIME_DIR=/run/user/$SLURM_JOB_UID"
```

使脚本可执行并添加到 `slurm.conf`：

```bash
sudo chmod 755 /etc/slurm/prolog.sh /etc/slurm/epilog.sh /etc/slurm/task_prolog.sh
```

添加到 `slurm.conf`：

```ini
Prolog=/etc/slurm/prolog.sh
Epilog=/etc/slurm/epilog.sh
TaskProlog=/etc/slurm/task_prolog.sh
```

更新 `slurm.conf` 后重启 Slurm 控制器：

```bash
sudo systemctl restart slurmctld
```

!!! warning "共享 HPC 节点上的 linger 模式"

    避免在没有额外加固的情况下在共享计算节点上使用 `loginctl enable-linger`。Linger 模式允许用户进程在作业完成后继续存在，这可能导致资源消耗不受限制、审计可见性降低以及进程超出 Slurm 作业记账范围。使用 cgroupfs 方法配合 prolog 和 epilog 脚本是推荐的替代方案。

## 故障排除

### 重启后 cgroup v2 未激活

验证内核参数是否已设置：

```bash
cat /proc/cmdline | grep unified
```

如果 `systemd.unified_cgroup_hierarchy=1` 未出现，grubby 命令可能未应用到当前内核。使用显式内核路径重新运行：

```bash
sudo grubby --update-kernel=/boot/vmlinuz-$(uname -r) --args="systemd.unified_cgroup_hierarchy=1"
```

### ssh 登录后 XDG_RUNTIME_DIR 未设置

验证 `pam_systemd.so` 是否已在 `/etc/pam.d/sshd` 中启用：

```bash
grep pam_systemd /etc/pam.d/sshd
```

如果该行存在但 `XDG_RUNTIME_DIR` 仍未设置，请重启 `systemd-logind` 并打开新的 ssh 会话：

```bash
sudo systemctl restart systemd-logind
```

不要使用 `su` 或 `sudo -i -u` 进行测试；它们不会触发 PAM 会话模块。

### 无根 Podman 出现 D-Bus 会话总线错误

如果您看到：

```text
dbus: couldn't determine address of session bus
Failed to create bus connection: No such file or directory
```

检查 `$DBUS_SESSION_BUS_ADDRESS` 是否已设置且总线套接字是否存在：

```bash
echo $DBUS_SESSION_BUS_ADDRESS
ls -la /run/user/$(id -u)/bus
```

如果总线套接字缺失，请按照 PAM 配置部分所述验证 `pam_systemd.so` 是否已启用。对于 HPC 计算节点，改用 cgroupfs cgroup 管理器，这不需要 D-Bus 会话。

### Slurm 作业失败并显示"Controller cpuset is not enabled"

这表示 cpuset 控制器在 Slurm cgroup 作用域中不可用。请验证：

```bash
cat /sys/fs/cgroup/cgroup.subtree_control
```

如果 `cpuset` 和 `cpu` 缺失，请按照上方的"在 systemd 239 上启用缺失的控制器"部分操作。确保 `/etc/systemd/system.conf` 中已设置 `DefaultCPUAccounting=yes` 且节点已重启。

同时验证 `/etc/slurm/cgroup.conf` 中已设置 `EnableControllers=yes` 且 `slurmctld` 已重启。

### Podman 回退到 cgroupfs 并显示警告

如果每个 Podman 命令都显示：

```text
WARN[0000] The cgroupv2 manager is set to systemd but there is no systemd user session available
WARN[0000] Falling back to --cgroup-manager=cgroupfs
```

这表示 Podman 找不到活跃的 systemd 用户会话。要么启用 `pam_systemd.so` 以获得适当的用户会话，要么在 containers.conf 中设置 `cgroup_manager = "cgroupfs"` 有意使用 cgroupfs 并抑制警告。

## 总结

在 Rocky Linux 8.10 上从 cgroup v1 迁移到 cgroup v2 需要解决 systemd 239 的限制。关键步骤是：

1. 通过内核引导参数启用统一层级
2. 使用 `DefaultCPUAccounting=yes` 启用缺失的控制器
3. 配置 PAM 以实现适当的用户会话（或在 HPC 环境中使用 prolog/epilog 脚本）
4. 为 Podman 设置适当的 cgroup 管理器
5. 使用 `EnableControllers=yes` 更新 Slurm cgroup 配置

对于 HPC 环境，使用 Slurm prolog 和 epilog 脚本的 cgroupfs 方法提供了一种不需要在共享计算节点上使用 systemd 用户会话或 linger 模式的稳健解决方案。

## 参考资料

1. "Control Group v2"，the Linux Kernel Team [https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)
2. "Control Group v2 plugin"，SchedMD [https://slurm.schedmd.com/cgroup_v2.html](https://slurm.schedmd.com/cgroup_v2.html)
3. "cgroup.conf - Slurm configuration file for the cgroup support"，SchedMD [https://slurm.schedmd.com/cgroup.conf.html](https://slurm.schedmd.com/cgroup.conf.html)
4. "libpod: report cgroups deleted during Stat() call"，Giuseppe Scrivano [https://github.com/containers/podman/pull/24400](https://github.com/containers/podman/pull/24400)
5. "systemd.resource-control - Resource control unit settings"，the systemd Team [https://www.freedesktop.org/software/systemd/man/latest/systemd.resource-control.html](https://www.freedesktop.org/software/systemd/man/latest/systemd.resource-control.html)
6. "pam_slurm_adopt"，SchedMD [https://slurm.schedmd.com/pam_slurm_adopt.html](https://slurm.schedmd.com/pam_slurm_adopt.html)
7. "Podman Documentation"，the Podman Team [https://docs.podman.io/en/latest/markdown/podman.1.html](https://docs.podman.io/en/latest/markdown/podman.1.html)
