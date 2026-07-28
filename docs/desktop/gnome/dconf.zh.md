---
title: dconf 配置编辑器
author: Ezequiel Bruni
contributors: Steven Spencer, Ganna Zhyrnova
---

## 介绍

GNOME 对其用户界面和功能采取了非常精简的方法。这并非不好，因为它易于学习，并且默认的 GNOME 体验让您可以开始做自己的工作。

然而，这种方法也意味着它不能快速配置。如果您在设置面板中找不到所需内容，可以安装 GNOME Tweaks 来扩展您的选项。您甚至可以安装 GNOME 扩展来获取新功能和选项。

但是，如果您想查看 GNOME 开发人员隐藏的所有小设置、功能和配置呢？您可以在线查找当前的问题，然后输入命令来更改一个晦涩的变量，或者您可以安装 `dconf Editor`。

`dconf Editor` 是一个*包含一切*的 GNOME 设置应用程序。它可能会让您联想到 Windows 注册表，并且确实类似。不过，它更具可读性，仅涵盖 GNOME 功能和一些为 GNOME 构建的软件。

您还可以编辑 GNOME 扩展的设置。

!!! warning

    与 Windows 注册表的比较是完全有意的。与错误的注册表键一样，*有些* GNOME Shell 设置可能会破坏您的 GNOME 安装，或至少导致问题。您可能需要通过命令行恢复旧设置。

    如果您不确定某个特定设置的作用，请先研究它。不过，更改应用程序设置是没问题的，而且更容易还原。

## 假设

对于本指南，您需要以下条件：

* 安装了 GNOME 的 Rocky Linux。
* 在系统上安装软件的权限（`sudo` 权限）。


## 安装 `dconf Editor`

转到"Software"应用程序，搜索"Dconf Editor"，点击安装按钮。它在默认的 Rocky Linux 仓库中可用。

![GNOME 软件中心，展示 dconf Editor](images/dconf-01.png)

要通过命令行安装 dconf Editor，请执行以下操作：

```bash
sudo dnf install dconf-editor
```

## 使用 `dconf Editor`

打开应用程序后，您将看到三个重要的用户界面部分。顶部是路径。所有 GNOME 设置都在一个路径/文件夹结构中。

在右上角，您会看到一个带有小星星的按钮。那是收藏夹按钮，它允许您保存您在应用程序中的位置，以便稍后快速轻松地返回。下方是中央面板，您可以在其中选择设置子文件夹，并随意更改设置。

![dconf Editor 窗口截图，箭头指向上述元素](images/dconf-02.png)

收藏夹按钮左侧是搜索按钮，其功能与您期望的完全一样。

如果您想更改文件管理器中的某些设置怎么办？例如，我喜欢侧边栏。我觉得它非常方便。但也许您有不同的感觉，想做一些改变。所以，对于本指南来说，它必须消失。

![Nautilus 文件管理器截图，带有威胁性的红色 X 标记在 doomed 侧边栏上](images/dconf-03.png)


转到 `/org/gnome/nautilus/window-state`，您将看到一个名为 `start-with-sidebar` 的选项。点击切换按钮，然后在出现时点击"Reload"按钮，如下截图所示：

![dconf Editor 截图，显示相关的切换和重新加载按钮](images/dconf-04.png)

如果一切顺利，您打开的下一个文件浏览器窗口应该如下所示：

![文件管理器截图，失去了侧边栏](images/dconf-05.png)

如果感觉不对，切换回来，再次点击 Reload，然后打开一个新的文件浏览器窗口。

最后，您可以直接点击 `dconf Editor` 窗口中的任何设置以查看更多信息（有时还有更多选项）。例如，下面是 GNOME 文件管理器的 `initial-size` 设置屏幕。

![dconf Editor 截图，显示文件管理器的 initial-size 设置](images/dconf-06.png)

## 故障排除

如果您在 `dconf Editor` 中更改设置后没有看到任何变化，请尝试以下修复之一：

1. 重新启动您正在更改的应用程序。
2. 注销、重新登录或重新启动以应用对 GNOME Shell 的某些更改。
3. 放弃，因为该选项不再起作用。

关于最后一点：是的，GNOME 开发人员有时会禁用您更改某个设置的能力，即使使用 `dconf Editor` 也是如此。

例如，我尝试更改窗口切换器设置（按下 ++alt+tab++ 时显示的打开窗口列表），但毫无进展。无论我尝试什么，`dconf Editor` 都不影响其某些功能。

这可能是一个 bug，但这不会是其设置显示在 `dconf Editor` 中但基本上被悄悄禁用的第一次。如果您遇到此问题，请搜索 GNOME Extensions 站点，查看是否有扩展可以添加您想要恢复到 GNOME 的选项。

## 总结

这就是您开始所需知道的一切。只需记得跟踪所有更改，不要在不知道确切作用的情况下更改设置，并享受探索（大部分）可用的选项。
