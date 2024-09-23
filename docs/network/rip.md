---
title: "rip"
layout: default
parent: Network
has_children: false
---

## 1. 基础信息

- rip是距离矢量协议，对于路由器来说并不清楚网络的拓扑，只知道去往目的网络的网段方向和开销

- 使用协议UDP，端口520, 使用组播地址224.0.0.9
![rip](/assets/images/network/rip-pro.png)

- 优先级是100
- rip 计时器，更新时间时30秒，
- 老化时间180秒
- 垃圾清理周期时120秒，**网络宣告需要按网络分类**
![rip-status](/assets/images/network/rip-status.png)


## 2. rip（v2）报文

- [华为参考地址](https://support.huawei.com/enterprise/zh/doc/EDOC1100262534/f64c41e3)

![报文](/assets/images/network/ripv2-报文.png)

## 3. 防环

- 接口配置检查使用`display rip process-id interface [ interface-type interface-number ] [ verbose ]`
![ripcheck](/assets/images/network/rip-check.png)

### 3.1 水平分割

- 配置命令(在接口视图下):`rip split-horizon`
- 水平分割的原理是，RIP从某个接口学到的路由，不会从该接口再发回给邻居路由器。这样不但减少了带宽消耗，还可以防止路由环路。
- 如果没有设置水平分割，路由器从某一个接口更新的路由信息，加1跳之后还会以此端口发送回去，两个路由器会不停的发送路由表更新信息，直到跳数达到峰值。设置水平分割之后可以有效的改善路由环问题


### 3.2 毒性反转

- 配置命令(在接口视图下):`rip poison-reverse`
- 毒性反转法的原理是，RIP从某个接口学到路由后，从原接口发回邻居路由器，并将该路由的开销设置为16（即指明该路由不可达）。利用这种方式，可以清除对方路由表中的无用路由。
- 当某一路由器发现与某直连网络断开以后，直接将此网络跳数设为16，并向相邻路由设备更新路由信息。使其都更新为16跳不可达。


## 4. 实验


- 操作和ospf相似，先配置路由器的接口地址，在配置rip协议
- 注意这里的1.1.1.1 在地址宣告的时候时1.0.0.0 时A类地址

![rip实验](/assets/images/network/rip-实验.png)

- 协议配置
```shell
rip
version 2
network 1.0.0.0
```

- 信息查看
    - 查看邻居：`disp rip 1 neighbor`
    - 查看数据库：`disp rip 1 database`
    - 查看协议路由表：`disp rip 1 route`
