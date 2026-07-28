---
title: QA:Testcase 去品牌化
author: Trevor Cooper
contributors:
tested_with: 8.10
tags:
  - testing
  - qa
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
  ver: 8
  level: Final
render_macros: true
---

!!! info "关联的发布标准"
    此测试用例关联 [Release_Criteria - Debranding](../../guidelines/release_criteria/r9/9_release_criteria.md#debranding) 发布标准。如果你正在进行发布验证测试，此测试用例的失败可能违反该发布标准。

## 描述

Rocky Linux [发布工程团队](https://sig-core.rocky.page/) 构建并维护工具来管理从上游供应商接收的软件包的去品牌化（debranding）。他们发布了一份全面的[去品牌化指南](https://sig-core.rocky.page/documentation/patching/debrand_info/)，并维护着一个需要去品牌化补丁的[软件包列表](https://git.rockylinux.org/rocky/metadata/-/blob/main/patch.yml)。

此测试用例将验证已发布介质上 Rocky Linux 发布工程确认为需要去品牌化的所有软件包均已按照其规范成功去品牌化。

## 设置

1. 获取对一个环境的访问权限，该环境具有 `dnf` 和 `koji` 命令，并能访问 [Rocky Linux Gitlab](https://git.rockylinux.org) 和 [Rocky Linux Koji](https://koji.rockylinux.org)。
2. 将要测试的 ISO 下载到测试机器。
3. 配置 `/etc/koji.conf` 以访问 [Rocky Linux Koji](https://koji.rockylinux.org)。
4. 从 [Rocky Linux Gitlab](https://git.rockylinux.org) 下载 [patch.yml](https://git.rockylinux.org/rocky/metadata/-/blob/main/patch.yml) 的最新副本。

!!! info "patch.yml"
    `patch.yml` 中列出的软件包是源代码 RPM（source RPM）的名称。需要验证由构建打了补丁的源代码 RPM 所产生的二进制 RPM 中包含的内容。获取特定软件包和架构的所有可能二进制 RPM 列表的最简单方法是在 koji 中获取这些信息。

## 如何测试

1. 将要测试的 ISO 挂载到本地。
    - 示例：

    ```bash
    mount -o loop Rocky-8.5-x86_64-dvd1.iso /media
    ```

2. 确定 ISO 上 `repodata` 目录的路径。
    - 示例：

    ```bash
    find /media -name repodata
    ```

3. 对于每个要验证的软件包，获取由它创建的 `noarch` 和 `<arch>` 特定软件包的名称。
    - 示例：

    ```bash
    koji --quiet latest-build --arch=x86_64 dist-rocky8-compose <package>
    koji --quiet latest-build --arch=noarch dist-rocky8-compose <package>
    ```

4. 使用 `dnf` 获取需要去品牌化的二进制软件包的路径。
    - 示例：

    ```bash
    $ dnf download --urls --repofrompath BaseOS,/media/BaseOS --repo BaseOS \
        --repofrompath Minimal,/media/Minimal --repo Minimal \
        <binary_package>
    ```

5. 从介质中复制 `<binary_package>` 并检查其元数据和/或内容，以确定它是否已明显被打了补丁。
    - 示例：

   ```bash
   $ rpm -q --changelog -p <path_to_binary_package> | head | \
   grep "Release Engineering <releng@rockylinux.org>" -C2 | \
   grep -Eq "<pattern_to_find>"

   $ rpm2cpio <path_to_binary_package> |
   cpio --quiet --extract --to-stdout .<file_to_examine> | \
   grep -Eq "<pattern_to_find>"
   ```

    !!! info "注意"
        请注意，并非所有去品牌化补丁都会直接修补文件并留下非常明显的痕迹，有些补丁甚至不添加 changelog 消息作为软件包已被打补丁或去品牌化的指示。有时唯一的解决方案是解压二进制软件包并直接检查内容，以找到某些可测试的内容。

6. 卸载 ISO。
    - 示例：

   ```bash
   umount /media
   ```

## 预期结果

1. 发布工程跟踪为需要去品牌化并发布在安装介质上的软件包确实按照其规范进行了去品牌化。

### 示例输出

=== "成功"

   ```bash
   $ sudo mount -o loop Rocky-8.5-aarch64-minimal.iso /media
   mount: /media: WARNING: device write-protected, mounted read-only.

   $ find /media -name repodata
   /media/BaseOS/repodata
   /media/Minimal/repodata

   $ curl -LOR https://git.rockylinux.org/rocky/metadata/-/raw/main/patch.yml
   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                  Dload  Upload   Total   Spent    Left  Speed
   100  3410  100  3410    0     0  20419      0 --:--:-- --:--:-- --:--:-- 20419

   $ yq .debrand.all[] patch.yml | column -x -c 100 -o " "
   abrt                   anaconda               anaconda-user-help   chrony
   cloud-init             cockpit                crash                dhcp
   dnf                    firefox                fwupd                gcc
   gcc-toolset-9-gcc      gcc-toolset-10-gcc     gcc-toolset-11-gcc   gcc-toolset-12-gcc
   gnome-settings-daemon  grub2                  httpd                initial-setup
   kernel                 kernel-rt              libdnf               libreoffice
   libreport              nginx                  opa-ff               opa-fm
   openscap               pesign                 PackageKit           python-pip
   python3                redhat-rpm-config      scap-security-guide  shim
   shim-unsigned-x64      shim-unsigned-aarch64  sos                  subscription-manager
   systemd                thunderbird            WALinuxAgent

   $ ./yq .debrand.r8[] patch.yml | column -x -c 100 -o " "
   dotnet3.0  fwupdate  gnome-boxes  libguestfs  pcs  plymouth
   python2

   注意：此示例中仅展示一个软件包。

   $ koji --quiet latest-build --arch=x86_64 dist-rocky8-compose sos

   $ koji --quiet latest-build --arch=noarch dist-rocky8-compose sos
   sos-4.1-9.el8_5.rocky.3.noarch
   sos-audit-4.1-9.el8_5.rocky.3.noarch

   $ dnf download --urls --repofrompath BaseOS,/media/BaseOS --repo BaseOS \
   --repofrompath Minimal,/media/Minimal --repo Minimal \
   sos sos-audit | grep -E "^file"
   file:///media/BaseOS/Packages/s/sos-4.1-5.el8.noarch.rpm
   file:///media/BaseOS/Packages/s/sos-audit-4.1-5.el8.noarch.rpm

   $ rpm -q --changelog -p /media/BaseOS/Packages/s/sos-4.1-5.el8.noarch.rpm | \
     head | grep "Release Engineering <releng@rockylinux.org>" -C2
   * Mon Oct 18 2021 Release Engineering <releng@rockylinux.org> - 4.1-5
   - Remove Red Hat branding from sos
   $ echo $?
   0

   $ rpm -q --changelog -p /media/BaseOS/Packages/s/sos-audit-4.1-5.el8.noarch.rpm | \
     head | grep "Release Engineering <releng@rockylinux.org>" -C2
   * Mon Oct 18 2021 Release Engineering <releng@rockylinux.org> - 4.1-5
   - Remove Red Hat branding from sos
   $ echo $?
   0

   $ umount /media
   ```

=== "失败"

   ```bash
   $ sudo mount -o loop Rocky-8.5-aarch64-minimal.iso /media
   mount: /media: WARNING: device write-protected, mounted read-only.

   $ find /media -name repodata
   /media/BaseOS/repodata
   /media/Minimal/repodata

   $ curl -LOR https://git.rockylinux.org/rocky/metadata/-/raw/main/patch.yml
   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                Dload  Upload   Total   Spent    Left  Speed
   100  3410  100  3410    0     0  20419      0 --:--:-- --:--:-- --:--:-- 20419

   $ yq .debrand.all[] patch.yml | column -x -c 100 -o " "
   abrt                   anaconda               anaconda-user-help   chrony
   cloud-init             cockpit                crash                dhcp
   dnf                    firefox                fwupd                gcc
   gcc-toolset-9-gcc      gcc-toolset-10-gcc     gcc-toolset-11-gcc   gcc-toolset-12-gcc
   gnome-settings-daemon  grub2                  httpd                initial-setup
   kernel                 kernel-rt              libdnf               libreoffice
   libreport              nginx                  opa-ff               opa-fm
   openscap               pesign                 PackageKit           python-pip
   python3                redhat-rpm-config      scap-security-guide  shim
   shim-unsigned-x64      shim-unsigned-aarch64  sos                  subscription-manager
   systemd                thunderbird            WALinuxAgent

   $ ./yq .debrand.r8[] patch.yml | column -x -c 100 -o " "
   dotnet3.0  fwupdate  gnome-boxes  libguestfs  pcs  plymouth
   python2

   注意：此示例中仅展示一个软件包。

   $ koji --quiet latest-build --arch=x86_64 dist-rocky8-compose sos

   $ koji --quiet latest-build --arch=noarch dist-rocky8-compose sos
   sos-4.1-9.el8_5.rocky.3.noarch
   sos-audit-4.1-9.el8_5.rocky.3.noarch

   $ dnf download --urls --repofrompath BaseOS,/media/BaseOS --repo BaseOS \
   --repofrompath Minimal,/media/Minimal --repo Minimal \
   sos sos-audit | grep -E "^file"
   file:///media/BaseOS/Packages/s/sos-4.1-5.el8.noarch.rpm
   file:///media/BaseOS/Packages/s/sos-audit-4.1-5.el8.noarch.rpm

   $ rpm -q --changelog -p /media/BaseOS/Packages/s/sos-4.1-5.el8.noarch.rpm | \
   head | grep "Release Engineering <releng@rockylinux.org>" -C2
   $ echo $?
   1

   $ rpm -q --changelog -p /media/BaseOS/Packages/s/sos-audit-4.1-5.el8.noarch.rpm | \
   head | grep "Release Engineering <releng@rockylinux.org>" -C2
   $ echo $?
   1

   $ umount /media
   ```

{% include 'teams/testing/qa_testcase_bottom.md' %}
