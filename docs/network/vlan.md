---
title: "vlan"
layout: default
parent: Network
has_children: false
---


## mux vlan

- 通过MUX VLAN提供的二层流量隔离的机制可以实现企业内部员工之间互相交流，而企业客户之间是隔离的。

- MUX VLAN分为Principal VLAN和Subordinate VLAN，Subordinate VLAN又分为Separate VLAN和Group VLAN。

- MUX VLAN划分表
![mux vlan](/assets/images/network/mux-vlan.png)

## super vlan
- super vlan 使用的是3层技术，节约了网段，同时隔离了广播域
- VLAN Aggregation（VLAN聚合，也称Super VLAN）技术就是在一个物理网络内，用多个VLAN隔离广播域，使不同的VLAN 属于同一个子网。


## QinQ

基于VLAN ID的灵活QinQ：为具有不同内层VLAN ID的报文添加外层Tag。
灵活QinQ功能是对基本QinQ功能的扩展，它比基本QinQ的功能更灵活，二者之间的主要区别是：

- 基本QinQ：对进入二层QinQ接口的所有帧都加上相同的外层Tag。

- 灵活QinQ：根据不同的内层Tag或符合流分类规则的帧，加上不同的外层Tag，对于用户VLAN的划分更加细致。


## vrrp

各备份组之间的虚拟IP地址不能重复。

保证同一备份组的两端设备上配置相同的备份组ID。

网络中存在多组备份组时，不同设备上的不同备份组的备份组ID不能重复，否则会导致虚拟MAC地址冲突。