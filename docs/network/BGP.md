---
title: "BGP"
layout: default
parent: Network
has_children: false
---


## 基础知识

- keepalive报文--用于维护BGP邻居关系，默认保持时间为60秒，老化时间为180秒。

- peer allow-as-loop命令用来配置本地AS号的重复次数。

- bgp（Path-Vector Routing Protoccol）:路径矢量路由协议

- AS,同一个组织管理下，使用同一选路策略的设备集合
    - AS分为私有和共有，16bit的AS号表示中64512-65534为私有，公有1-64511
    - 32bit长度中4200000000-4294967294为私有
- 使用TCP端口179


- AS 之间需要直连链路，或者通过VPN协议构造逻辑直连，建立邻居
- BGP是触发更新，不是周期更新

- 现在bgp版本是4如果是ipv6 则是bgp4+

- bgp speaker：运行bgp的路由器
- bgp只发送增量的路由更新，或者出发式更新（不会周期的更新）


- **对等体**
    - IBGP：位于AS内的路由器之间， **建议使用环回口建立邻居关系**
    - EBGP： 位于不同AS之间的对等体， **建立使用直连接口**


## bgp报文

- open报文携带的
    - My Autonomous System: 自身的AS号
    - Hold Time:协商后续keepalive的报文发送时间,**默认keepalive是60秒，HoldTime是Keep的三倍就是180秒**，如果两者时间不一致，则选择时间小的
    - BGP Identifier: 自身的Router ID

    - 在open报文断掉以后需要等到三倍的时间才会断掉邻居关系

- Update ：报文更新路由到对等体
- Notification ： 报告错误信息，中止对等体关系
- Keepalive: 标志对等体建立，维持BGP对等体关系
- Route-refresh: 用于在改变路由策略后请求对等体重新发送路由信息


- 路由注入：
    - network （只能通告路由表中存在的）
    - import-route （`import-route ospf`,`import-route isis`,`import-route static`,`import-route direct`）

- 路由聚合`bgp 200; aggregate x.x.x.x detail-suppressed` detail-suppressed：不收明细

- 通告原则：
    - 只发布最优路由
    - 从EBGP对等体获取的路由，会发布给所有对等体
    - IBGP水平分割：从IBGP对等体获取到的路由，不会发送给IBGP对等体
    - BGP同步规则：从IBGP对等体学习到的路由，不会通告给EBGP除非，又从IGP（ospf等）学习到了这条路由

- 路径属性
    - 公认必遵：`origin，AS_Path,Next_hop`              所有bgp路由器必须能够识别的属性，必须包含在Update消息里
    - 公认任意：`Local_Preference,Atomic_aggregate`     可能包含在Update消息里
    - 可选过度：`Aggregator,Community`                  BGP设备不识别此类属性依然接受并通告给其他对等体
    - 可选任意：`MED,Cluster-List,Originator-ID`        BGP设备不识别此类属性会忽略，且不会通告给其他对等体

- MED：是进入AS的最佳路径

## 路由反射器

- NLRI(可达路由前缀，可以理解为路由)

- 解决IBGP全互联，引入RR（route Reflector）路由反射器
- RR在收到BGP路由时
    - 如果反射器从自己的非客户端对等体收到IBGP路由，则将路由反射给所有客户
    - 如果反射器从自己的客户学习到一条IBGP路由，则反射给所有非客户和客户，但排除发送给自己的客户端
    - 如果发射器从EBGP对等体学习到则发送给所有客户和非客户IBGP对等体

- 路由反射器使用何种属性防环？
    - 内部：使用Originator-ID
    - 之间：Cluster-List

- MP-BGP新增两种属性，都是（可选非过度）
    - MP_REACH_NLRI: 目标可达，通告下一条的可达，
    - MP_UNREACH_NLRI：目标不可达， 撤销下一跳的可达

- VPLS(Virtual Private LAN Service) 二层vpn技术，它在mpls网络上提供了类似lan的业务，允许用户从多个地址接入，相互访问，**缺点：需要arp广播泛洪，来学习远端的mac**
- EVPN：控制平面采用MP-BGP,数据平面采用MPLS，**类似SDN,实现转控分离** 

- EVPN
