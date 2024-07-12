---
title: "Linux virtual network"
permalink: /pages/network/virt-network
excerpt: "Linux 下常见的虚拟网络介绍以及Linux的路由和路由表介绍."
toc: true
excerpt_separator: <!--more-->
tags: linux network virt veth route
---

Linux 下常见的虚拟网络介绍以及Linux的路由和路由表介绍
<!--more-->

# 常见的虚拟网络设备
## veth
veth pair是一对虚拟网络设备，一端发送的数据会由另外一端接受，常用于不同的网络命名空间
虚拟网卡对的流量会从一端传到另一端,[veth说明](https://man7.org/linux/man-pages/man4/veth.4.html).

使用ip命令可以轻松的创建一个虚拟网卡对,比如:
```shell
ip link add veth0 type veth peer name veth00
```
## tap/tun


TAP/TUN设备是一种让用户态程序向内核协议栈注入数据的设备，TAP等同于一个以太网设备，工作在二层；而TUN则是一个虚拟点对点设备，工作在三层。

## macvlan/ipvlan

通过ip或者mac来过滤流量
常用于添加underlay网络

## bridge

虚拟网桥，实现二层转发
可以通过brctl 命令来创建和管理，可以通过ip命令

## vrf

VRF全称是虚拟路由转发（Virtual Routing Forwarding）
虚拟隔离路由，实现三层路由

# 网络路由

## 路由规则

在发送ip包之前会把发送到目的ip与本地路由表进行比较来判断从哪个网卡出去


# iptables

iptables 可以检测、修改、转发、重定向和丢弃 IPv4 数据包。
iptables 区分ipv4和ipv6,v4主要用命令`iptables`,v6用命令`ip6tables`

