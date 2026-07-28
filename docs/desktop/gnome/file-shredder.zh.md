---
title: File Shredder - 安全删除
author: Wale Soyinka
contributors: Ganna Zhyrnova
tags:
  - gnome
  - desktop
  - security
  - privacy
  - flatpak
---

## 永久且安全地删除文件

当您使用文件管理器删除文件时，数据并不会被真正擦除。操作系统只是将硬盘上的空间标记为"可用"，原始数据在被新文件覆盖之前一直保留在原位。这意味着"已删除"的文件通常可以通过专业软件恢复。

**File Shredder** 是一款面向 GNOME 桌面的现代简洁工具，通过反复用随机数据覆盖文件数据再执行删除，使得恢复几乎不可能，从而解决了这一问题。

## 安装

在 Rocky Linux 上推荐的安装方式是使用 Flathub 仓库提供的 Flatpak。

### 1. 启用 Flathub

如果您尚未完成此操作，请确保系统已安装 Flatpak 并配置了 Flathub 远程仓库。

```bash
# Install the Flatpak package
sudo dnf install flatpak

# Add the Flathub remote repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

### 2. 安装 File Shredder

Flathub 启用后，只需一行命令即可安装 File Shredder。

!!! note "Application ID"
    该应用程序名为 "File Shredder"，但它在 Flathub 上的技术 ID 是 `io.github.ADBeveridge.Raider`。

```bash
flatpak install flathub io.github.ADBeveridge.Raider
```

## 如何使用 File Shredder

安装完成后，从 GNOME "活动概览"(Activities Overview) 搜索 "File Shredder" 即可启动。

操作流程简单明了：

1.  将您希望永久删除的文件或文件夹直接拖放到 File Shredder 窗口中。也可以点击 **"Add Files..."** 或 **"Add Folder..."** 按钮。
2.  文件将出现在列表中。请仔细检查此列表。
3.  当您确认要永久销毁这些文件时，点击 **Shred** 按钮。

!!! warning "此操作不可逆"
    没有"撤销"按钮。文件一经粉碎，便永久消失。在点击 Shred 按钮之前，请再三确认已添加的文件。

## SSD 的重要注意事项

虽然 File Shredder 对传统磁性硬盘 (HDD) 非常有效，但在现代固态硬盘 (SSD) 上的效果存在局限性。

SSD 使用复杂的内部机制（如磨损均衡和垃圾回收）来管理数据并延长驱动器寿命。这些机制意味着是驱动器本身（而非操作系统）决定数据物理写入的位置。像 File Shredder 这样的软件工具无法强制 SSD 覆盖特定的物理数据块。

因此，虽然在 SSD 上使用 File Shredder 使数据恢复比普通删除困难得多，但它**无法保证**数据的全部痕迹已从驱动器存储单元中物理擦除。在 SSD 上实现最高级别的数据安全，推荐的方法是使用全盘加密（例如可在 Rocky Linux 安装过程中设置的 LUKS）。

File Shredder 仍然是增强数据隐私的重要工具，特别是在 HDD 上，并为大多数使用场景提供了强有力的安全层。
