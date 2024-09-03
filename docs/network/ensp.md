---
title: "ensp"
layout: default
parent: Network
has_children: false
---


## 常用名字解释

- MPU: main processing unit 主处理单元
- SFU: Switch fabric unit 交换结构单元
- LPU: line processing unit 线路处理单元
- CMU: entralized Monitoring Unit 集中监控单元
- RIB: routing information base 路由信息库（在控制器平面）
- FIB: forwarding information base 转发信息库 (在数据平面)
- IGP: interior gateway protocol: 在一个自治系统（AS）内部运行（常见的IGP协议包括OSPF、IS-IS）
- EGP: exterior gateway protocol: 运行在不同的自治系统(AS)之间（BGP是最常见的EGP协议）
- OSPF: Open Shortest Path First: 开放式最短路径优先
 - DR （Designated Router）：多接入网络中的指定路由器
 - BDR： 备份指定路由器
 - IR（Internal Router）：区域内路由器，所以接口都在一个区域内。
 - ABR（Area Border Router）：区域边界路由器，接口属于两个区域以上，但必须有一个连接骨干区域
 - BR（Backbone Router）：骨干路由器，至少有一个接口属于骨干区域。
 - ASBR（AS Boundary Router）：自治系统边界路由器，该类路由器与其他AS交换路由信息。只要一台OSPF路由器引入了外部路由的信息，它就成为ASBR。
- LSDB: Link State DataBase: 链路状态数据库
- LSA: Link-State Advertisement: 链路状态通告
 - LSA Type 1 - Router LSA（路由器LSA）：设备自身的链路状态和开销
 - LSA Type 2 - Network LSA（网络LSA）： 由DR产生，秒速MA（多路访问）网络的具体情况
 - LSA Type 3 - Network Summary LSA（汇总LSA）：由ABR（Area Border Router）产生。
 - LSA Type 4 - ASBR-Summary LSA（ASBR汇总LSA）：ASBR-Summary LSA是由ASBR（AS Boundary Router）产生的。
 - LSA Type 5 - AS-External-LSA（AS外部LSA）AS-External-LSA同样也是由ASBR（AS Boundary Router）产生的。
 - LSA Type 7 - NSSA LSA（NSSA LSA）：由NSSA区域或Totally NSSA区域的NSSA ASBR产生。

- 路由迭代：路由必须有直连的下一跳才能指导转发，静态路由或者BGP路由的下一跳可能不是直连的邻居，因此需要计算出一个直连的下一跳，这个过程叫做路由迭代
- 路由引入：将一个路由的信息从一种路由协议发布到另一种路由协议
 - 比如将A的路由引入到B这需要再B上操作，这个时候B上引入的A的路由则称为外部路由，这个外部路由的优先级是255是很低的，这个时候如果和内部路由冲突优先是走内部
 - 路由回灌： 
 - 次优路由：
 - 路由回路：
 - 路由引入以后开销需要修改`import-route isis 1 cost 333`



## ensp 常用命令
- 获取当前配置：`disp cu`
- 配置静态路由`ip route-static 172.16.10.0 255.255.255.0 10.10.12.1`，表示172.16.10.0/24 通过10.10.12.1 到达
- 查看路由表`disp ip routing-table`简写`disp ip ro`
- 查看具体的路由信息`ip routing-table 192.168.1.0 verbose`
- 查看ospf路由`disp ospf routing`
- 查看当前接口的配置`disp this`




- 初始化配置ip
```shell
[Huawei]sysname AR4 //命名
[AR4]interface GigabitEthernet 0/0/1 //开始配置
[AR4-GigabitEthernet0/0/1]ip address 10.10.14.4 24
[AR4-GigabitEthernet0/0/1]quit
[AR4]interface GigabitEthernet 0/0/2
[AR4-GigabitEthernet0/0/2]ip address 10.10.34.4 24
```


## ospf实验
- 查看lsdb`disp ospf lsdb`
- 区域md5认证`authentication-mode md5 1 plain HUAWEI`

```shell
# 初始化配置设备
# 进入ospf
ospf
# 进去区域
area 1
# 给设备配置网络，所有的网络都需要配置
net 10.1.13.1 0.0.0.0
```
### stub 和 nssa

- stub配置是进入area以后直接`stub`命令就行
- totally stub 则是在ABR上对应的区域执行`stub no-summary`


- nssa配置是进入area以后直接`nssa`命令就行
- totally nssa 则是在ABR上对应的区域执行`nssa no-summary`