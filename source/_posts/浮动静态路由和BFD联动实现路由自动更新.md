---
title: 浮动静态路由和BFD联动实现路由自动更新
top: false
cover: false
toc: true
mathjax: true
date: 2019-11-16 11:49:29
password:
summary: 浮动静态路由是一种特殊的静态路由
tags: Linux网络运维
categories: Linux
---

#### 一、浮动静态路由和BFD简介

浮动静态路由是一种特殊的静态路由，通过配置一个比主路由的管理距离更大的静态路由，保证网络中主路由失效的情况下，提供备份路由。

BFD是Bidirectional Forwarding Detection的缩写，它是一个用于检测两个转发点之间故障的网络协议，是一种双向转发检测机制，可以提供毫秒级的检测，可以实现链路的快速检测，BFD通过与上层路由协议联动，可以实现路由的快速收敛，确保业务的永续性。

#### 二、实验拓扑图 

![拓扑图](tuoputu.png)

#### 三、配置信息

##### 1、AR1配置

```cpp
[Huawei]vlan 10
[Huawei-vlan10]q
[Huawei]int g0/0/0
[Huawei-GigabitEthernet0/0/0]ip add 172.16.0.1 24
[Huawei-GigabitEthernet0/0/0]q
[Huawei]int g0/0/1
[Huawei-GigabitEthernet0/0/1]ip add 172.16.2.1 24
[Huawei-GigabitEthernet0/0/1]q
[Huawei]int g0/0/2
[Huawei-GigabitEthernet0/0/2]ip add 192.168.0.1 24
```

##### 2、AR2配置

```cpp
<Huawei>sys
[Huawei]iint g0/0/0
[Huawei]int g0/0/0
[Huawei-GigabitEthernet0/0/0]ip add 172.16.0.2 24
[Huawei-GigabitEthernet0/0/0]q
[Huawei]int g0/0/1
[Huawei-GigabitEthernet0/0/1]ip add 172.16.1.1 24
[Huawei-GigabitEthernet0/0/1]q
[Huawei]
```

##### 3、AR3配置

```cpp
<Huawei>sys
[Huawei]int g0/0/0
[Huawei-GigabitEthernet0/0/0]ip add 172.16.1.2 24
[Huawei-GigabitEthernet0/0/0]
[Huawei-GigabitEthernet0/0/0]q
[Huawei]int g0/0/1
[Huawei-GigabitEthernet0/0/1]ip add 172.16.2.2 24
[Huawei-GigabitEthernet0/0/1]q
[Huawei]int g0/0/2
[Huawei-GigabitEthernet0/0/2]ip add 192.168.1.1 24
[Huawei-GigabitEthernet0/0/2]q
[Huawei]
```

##### 4、BFD联动配置

###### 1. AR1

```cpp
[AR1]bfd

[AR1] quit

[AR1] bfd tobeijing peer-ip 172.16.2.2

[AR1-bfd-session-tobeijing] discriminator local 10

[AR1-bfd-session-tobeijing] discriminator remote 20

[AR1-bfd-session-tobeijing]commit

[AR1-bfd-session-tobeijing]quit

[AR1]undo ip route-static 192.168.1.0 24 172.16.2.2

[AR1]ip route-static 192.168.1.0 24 172.16.2.2 track bfd-session tobeijing
```

###### 2. AR3

```cpp
[AR3]bfd

[AR3] quit

[AR3] bfd toshanghai peer-ip 172.16.2.1

[AR3-bfd-session-toshanghai] discriminator local 20

[AR3-bfd-session-toshanghai] discriminator remote 10

[AR3-bfd-session-toshanghai]commit

[AR3-bfd-session-toshanghai]quit

[AR3]undo ip route-static 192.168.0.0 24 172.16.2.1

[AR3]ip route-static 192.168.0.0 24 172.16.2.1 track bfd-session toshanghai
```
