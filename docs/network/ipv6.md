---
title: "ipv6"
layout: default
parent: Network
has_children: false
---


# 基础

## ipv6表示方式

- ipv6 使用冒号分割16进制的方式表达一个地址比如：`fe80:0:0:0:20c:f1ff:fefd:d2be`
- ipv6 使用两个冒号省略中间的0比如上面的ipv6可以简写为：`fe80::20c:f1ff:fefd:d2be`
    - 比如本地回环地址 `::1 = 0000:0000:0000:0000:0000:0000:0000:0001`
    - 将ipv4藏入ipv6,使用ipv6的最后四位换成ipv4 `::ffff:127.0.0.1 = 0000:0000:0000:0000:0000:ffff:7f00:0001`

- ipv6 使用与ipv4一样的方式使用末尾的斜线加十进制的方式表达一个网段比如：`fe80::/64`
- 值得注意的是ipv6的所有单播地址通常都是64位的，也就是主机位和网络位55开整个128位长度
- ipv4 中的私有地址A类`10.0.0.0/8`,B类`172.16.0.0/12` C类`192.168.0.0/16`


## Scopes

- ipv6 废除了NAT因为数量足够多不需要NAT
- 虽然iv6废除了NAT但是保留了这种思想，也就是scopes

    - Global scope: 全球唯一地址
    - Site-local scope: 站点地址，一般是`fd00::/8`,或者`fc00::/8`
    - Link-local scope：本地链路地址，不会被路由比如：`fe80::/64`



### global scope

- **单播地址**
    - ![glabel scope](/assets/images/network/global-ipv6.jpg)

    - 全球地址前48位用于预路由地址和16位的站点子网表示和64位的主位，够成了一个全球的唯一单播地址
    - `2002::/16`这些地址用于“6to4隧道”。他们提供了一个无配置的仅通过IPv4运行通往Internet6的隧道


- **组播地址** 
    - ![multicast ipv6](/assets/images/network/multicast-ipv6.png)

    - 组播使用`ff00::/8`开头
    - 然后flag:整个4位是0或者是f表示是临时地址还是永久地址
    - scope标识位如下，(0和3是保留没使用的)
        - 1 interface-local
        - 2 link-local
        - 4 admin-local
        - 5 site-local
        - 6 (unnamed)
        - 7 (unnamed)
        - 8 organization-local
        - 9 (unnamed)
        - a (unnamed)
        - b (unnamed)
        - c (unnamed)
        - d (unnamed)
        - e global
    - 剩下的就是主播组的标识
    - 两个特殊的组播地址
        - `ff02::1`:节点本地链路组播地址,ping所有本地地址`ping -6 -I enp0s31f6 ff02::1`
        - `ff02::2`:路由本地组播地址,`ping -6 -I enp0s31f6 ff02::2`

- **广播地址** 


