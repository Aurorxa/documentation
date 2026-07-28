---
title: iftop - 实时逐连接带宽统计
author: Neel Chauhan
contributors: Ganna Zhyrnova, Steven Spencer
date: 2024-02-24
---

# `iftop` 简介

`iftop` 是一个基于文本控制台的网络流量监控器。它显示您服务器的逐连接流量和带宽统计信息。

## 使用 `iftop`

```bash
dnf -y install epel-release
dnf -y install iftop
```

`iftop` 命令的选项如下。

|选项|描述|
|---|---|
|-n |避免主机名查找|
|-N |避免将端口号解析为服务名称|
|-p |以混杂模式运行，统计所有流量|
|-P |显示连接的端口号|
|-l |显示并统计进出 link-local IPv6 地址的流量|
|-b |不显示流量条形图|
|-m LIMIT |设置带宽图的上限，以数字和单位后缀指定|
|-u UNIT |以给定单位显示流量速率|
|-B UNIT |与 -u 同义|
|-i INTERFACE |要监控的网络接口|
|-f FILTER CODE |使用以下过滤代码|
|-F NET/MASK |仅监控到指定 IPv4 网络的流量|
|-G NET/MASK |仅监控到指定 IPv6 网络的流量|
|-c config |使用以下配置文件|
|-t |使用非 ncurses 模式|

**-M** 标志的单位如下：

|单位|含义|
|---|---|
|K |千（Kilo）|
|M |兆（Mega）|
|G |吉（Giga）|

**-u** 标志的单位如下：

|单位|含义|
|---|---|
|bit |比特每秒|
|bytes |字节每秒|
|packets |数据包每秒|

作者家庭服务器运行 [Tor](https://www.torproject.org/) [中继](https://community.torproject.org/relay/types-of-relays/) 的示例输出：

```bash
 Listening on bridge b          25.0Kb          37.5Kb          50.0Kb    62.5Kb
└───────────────┴───────────────┴───────────────┴───────────────┴───────────────
tt.neelc.org               => X.X.X.X                    13.5Mb  13.5Mb  13.5Mb
                           <=                             749Kb   749Kb   749Kb
tt.neelc.org               => X.X.X.X                    6.21Mb  6.21Mb  6.21Mb
                           <=                             317Kb   317Kb   317Kb
tt.neelc.org               => X.X.X.X                    3.61Mb  3.61Mb  3.61Mb
                           <=                             194Kb   194Kb   194Kb
tt.neelc.org               => X.X.X.X                     181Kb   181Kb   181Kb
                           <=                            3.36Mb  3.36Mb  3.36Mb
tt.neelc.org               => X.X.X.X                     151Kb   151Kb   151Kb
                           <=                            3.24Mb  3.24Mb  3.24Mb
tt.neelc.org               => X.X.X.X                    2.97Mb  2.97Mb  2.97Mb
                           <=                             205Kb   205Kb   205Kb
tt.neelc.org               => X.X.X.X                     156Kb   156Kb   156Kb
                           <=                            2.97Mb  2.97Mb  2.97Mb
tt.neelc.org               => X.X.X.X                    2.80Mb  2.80Mb  2.80Mb
                           <=                             145Kb   145Kb   145Kb
tt.neelc.org               => X.X.X.X                     136Kb   136Kb   136Kb
                           <=                            2.45Mb  2.45Mb  2.45Mb
────────────────────────────────────────────────────────────────────────────────
TX:             cum:   30.1MB   peak:    121Mb  rates:    121Mb   121Mb   121Mb
RX:                    30.4MB            122Mb            122Mb   122Mb   122Mb
TOTAL:                 60.5MB            242Mb            242Mb   242Mb   242Mb
```

底部窗格各行的含义：

* TX - 发送/上传数据使用量
* RX - 接收/下载数据使用量
* TOTAL - 上传和下载合计使用量

## 交互快捷键

* ++s++ - 按每个源聚合所有流量
* ++d++ - 按每个目标聚合所有流量
* ++shift+s++ - 切换源端口的显示
* ++shift+d++ - 切换目标端口的显示
* ++t++ - 在显示模式之间切换：默认的两行显示（发送和接收流量）和三行显示（发送、接收和总流量）
* ++1++、++2++、++3++ - 按第 1、第 2 或第 3 列排序
* ++l++ - 输入 POSIX 正则表达式来过滤主机名
* ++shift+p++ - 暂停当前显示
* ++o++ - 冻结总带宽计数
* ++j++ - 向下滚动
* ++k++ - 向上滚动
* ++f++ - 编辑过滤代码
