---
title: QA:Testcase Vagrant 镜像
author: Bob Robison
contributors:
tested_with: 8.10, 9.7, 10.1
tags:
  - testing
  - qa
revision_date: 2026-05-08
rc:
  prod: Rocky Linux
render_macros: true
---

本页面提供有关如何启动/测试 Vagrant 镜像的信息。

## BIOS 启动的 Vagrant 文件

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "custom/rocky10u0-official"
  #config.vm.box = "rockylinux/10"
  config.vm.hostname = "rockylinux10"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.boot_timeout = 7200

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
    vb.cpus = "2"

    vb.customize ["modifyvm", :id, "--vram", "32"]
    vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]

    # 让 Vagrant 知道客户机没有安装客户机增强功能 (guest additions)，也没有可用的 vboxsf 或共享文件夹。
    vb.check_guest_additions = false
    vb.functional_vboxsf = false
  end

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    cat /etc/rocky-release
    mokutil --sb-state
  SHELL

end
```

## Vagrant BIOS 启动示例

```bash
~/boxes/official/rocky10u0 ❯ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'custom/rocky10u0-official'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: rocky10u0_default_1749351226445_30105
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Setting hostname...
==> default: Running provisioner: shell...
    default: Running: inline script
    default: Rocky Linux release 10.0 (Red Quartz)
    default: EFI variables are not supported on this system
The SSH command responded with a non-zero exit status. Vagrant
assumes that this means the command failed. The output for this command
should be in the log above. Please read the output to determine what
went wrong.

~/boxes/official/rocky10u0 ❯ vagrant ssh
[vagrant@rockylinux10 ~]$ sudo coredumpctl list
No coredumps found.
```

## UEFI 启动的 Vagrant 文件

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "custom/rocky10u0-official"
  #config.vm.box = "rockylinux/10"
  config.vm.hostname = "rockylinux10"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.boot_timeout = 7200

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
    vb.cpus = "2"

    # 以 EFI 模式启动机器
    vb.customize ["modifyvm", :id, "--firmware", "efi64"]

    vb.customize ["modifyvm", :id, "--vram", "32"]
    vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]

    # 让 Vagrant 知道客户机没有安装客户机增强功能 (guest additions)，也没有可用的 vboxsf 或共享文件夹。
    vb.check_guest_additions = false
    vb.functional_vboxsf = false
  end

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    cat /etc/rocky-release
    mokutil --sb-state
  SHELL

end
```

## Vagrant UEFI 启动示例

```bash
~/boxes/official/rocky10u0 ❯ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'custom/rocky10u0-official'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: rocky10u0_default_1749353062773_30641
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Setting hostname...
==> default: Running provisioner: shell...
    default: Running: inline script
    default: Rocky Linux release 10.0 (Red Quartz)
    default: SecureBoot disabled
    default: Platform is in Setup Mode


~/boxes/official/rocky10u0 ❯ vagrant ssh
[vagrant@rockylinux10 ~]$ sudo coredumpctl list
No coredumps found.
```

## UEFI 安全启动的 Vagrant 文件

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "custom/rocky10u0-official"
  #config.vm.box = "rockylinux/10"
  config.vm.hostname = "rockylinux10"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.boot_timeout = 7200

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
    vb.cpus = "2"

    # 以 EFI 模式启动机器
    vb.customize ["modifyvm", :id, "--firmware", "efi64"]

    # 启用安全启动 (Secure Boot) 启动机器
    vb.customize ["modifyvm", :id, "‑‑tpm‑type", "2.0"]
    vb.customize ["modifynvram", :id, "inituefivarstore"]
    vb.customize ["modifynvram", :id, "enrollmssignatures"]
    vb.customize ["modifynvram", :id, "enrollorclpk"]
    vb.customize ["modifynvram", :id, "secureboot", "--enable"]

    vb.customize ["modifyvm", :id, "--vram", "32"]
    vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]

    # 让 Vagrant 知道客户机没有安装客户机增强功能 (guest additions)，也没有可用的 vboxsf 或共享文件夹。
    vb.check_guest_additions = false
    vb.functional_vboxsf = false
  end

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    cat /etc/rocky-release
    mokutil --sb-state
  SHELL

end
```

## Vagrant UEFI 安全启动示例

```bash
~/boxes/official/rocky10u0 ❯ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'custom/rocky10u0-official'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: rocky10u0_default_1749353286744_54694
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Setting hostname...
==> default: Running provisioner: shell...
    default: Running: inline script
    default: Rocky Linux release 10.0 (Red Quartz)
    default: SecureBoot enabled

~/boxes/official/rocky10u0 ❯ vagrant ssh
[vagrant@rockylinux10 ~]$ sudo coredumpctl list
No coredumps found.
```
