---
title: nload - 带宽统计
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova 
date: 2024-01-16
---

# `nload` 简介

`nload` 是一个基于文本控制台的网络流量监控器。它显示您服务器的流量和带宽统计信息。

## 使用 `nload`

```bash
dnf -y install epel-release
dnf -y install nload
```

`nload` 命令的常用选项如下，在正常情况下无需其他额外的配置。选项放在要监控的网络接口之前：

|选项|描述|
|---|---|
|-a PERIOD |计算时间窗口的长度，以秒为单位（默认：300）|
|-m |显示多个设备但不显示流量图|
|-t INTERVAL |刷新间隔，以毫秒为单位（默认：500）|
|-u UNIT |带宽显示的单字母单位（默认：k）|
|-U UNIT |数据传输显示的单字母单位（默认：M）|

后两个选项的单位如下：

|单位|含义|
|---|---|
|b |bit（比特）|
|B |byte（字节）|
|k |kilobit（千比特）|
|K |kilobyte（千字节）|
|m |megabit（兆比特）|
|M |megabyte（兆字节）|
|g |gigabit（吉比特）|
|G |gigabyte（吉字节）|

作者家庭服务器运行 [Tor](https://www.torproject.org/) [中继](https://community.torproject.org/relay/types-of-relays/) 的示例输出：

```bash
Device bridge0 [172.20.0.3] (1/8):
================================================================================
Incoming:
                                             ########
                                             ########
                                             ########
                                             ########
                                             ########
                                             ########  Curr: 79.13 MBit/s
                                             ########  Avg: 84.99 MBit/s
                                             ########  Min: 79.13 MBit/s
                                             ########  Max: 87.81 MBit/s
                                             ########  Ttl: 1732.95 GByte
Outgoing:
                                             ########
                                             ########
                                             ########
                                             ########
                                             ########
                                             ########  Curr: 81.30 MBit/s
                                             ########  Avg: 88.05 MBit/s
                                             ########  Min: 81.30 MBit/s
                                             ########  Max: 91.36 MBit/s
                                             ########  Ttl: 1790.74 GByte
```

分解以上各行：

* Curr - 当前测量的带宽使用量
* Avg - 时间段内的平均带宽使用量
* Min - 测量到的最小带宽使用量
* Max - 测量到的最大带宽使用量
* Ttl - `nload` 会话中传输的数据总量

## 交互快捷键

* ++page-down++、++down++ - 切换到下一个网络接口
* ++page-up++、++up++ - 切换到上一个网络接口
* ++f2++ - 显示选项窗口
* ++f5++ - 保存选项
* ++f6++ - 从配置文件重新加载设置
* ++q++、++ctrl+c++ - 退出 `nload`
