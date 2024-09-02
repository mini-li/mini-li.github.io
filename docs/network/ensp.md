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
- 查看ospf路由`disp ospf rou`

- 初始化配置ip
```shell
[Huawei]sysname AR4 //命名
[AR4]interface GigabitEthernet 0/0/1 //开始配置
[AR4-GigabitEthernet0/0/1]ip address 10.10.14.4 24
[AR4-GigabitEthernet0/0/1]quit
[AR4]interface GigabitEthernet 0/0/2
[AR4-GigabitEthernet0/0/2]ip address 10.10.34.4 24
```
