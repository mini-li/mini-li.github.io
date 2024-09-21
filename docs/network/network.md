---
layout: default
title: Network
nav_order: 3
has_children: true
permalink: docs/network
---




- NAT:NetWork Address Transfer网络地址转化
 - 一个全局IP对应一个私有IP（即一台计算机）（或者有一个IP地址池，里面有很多外部IP地址，当有内部IP地址要出去的时候，就随机选一IP地址作为其出口IP）。一对一，一个内部地址转换成一个外部地址进行通信。
- NAPT: Network Address Port Transfer 网络地址端口转换。也叫PAT
 - 一个全局IP+不同的端口号对应多个私有IP（即多台计算机），需要一个全局IP对应多台计算时，比如局域网内部计算机访问外界网络时，就得用NAPT。一对多，多个内部地址使用同一地址不同端口转换成外部地址进行通信。缺点在于其通信仅限于TCP或UDP。
