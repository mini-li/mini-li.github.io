---
title: "ensp"
layout: default
parent: Network
has_children: false
---


## 尝试
- 交换机链路优先级，系统LACP优先级值越小优先级越高，缺省情况下，系统LACP优先级为32768。


## 常见协议及使用的端口
- LDP：Discovery（Hello）消息使用UDP（端口646）LDP的Session消息、Advertisement消息和Notification消息都使用TCP（端口646）。
- RADIUS: 用UDP（User Datagram Protocol）的RADIUS报文格式及其传输机制，并规定UDP端口1812、1813分别作为默认的认证、计费端口


## 常用名字解释
- NDP （icmpv6）: type
 - 133 RS路由请求
 - 134 路由通告
 - 135 邻居请求
 - 136 邻居通告
 - 137 重定向请求

- CSS （cluster Switch system） 集群
- iStack 堆叠
 - 堆叠以后没有环路无需stp和vrrp
- MAD （Multi-Active Detection） 多主检测
 - 直连检测和代理检测，两种互斥只能选择一种

- VRRP（Virtual Router Redundancy Protocol）虚拟路由冗余协议是：一种用于提高网络可靠性的容错协议
- DSCP（Differentiated Services Codepoint）：ip头中的第二个字节
- MPLS域： 一系列连续的运行MPLS的网络设备构成的一个MPLS域
 - LDP（Label Distribution Protocol）：标签转发协议,动态LSP
  - FTN (FEC TO NHLFE): 当LSR收到IP报文并需要进行MPLS转发时使用，FTN只存在于Ingress中，FTN包括，Tunnel ID，FEC到NHLFE的映射信息
  - NHLFE （Next Hop Label Forwarding Entry， 下一跳标签转发项）LSR对报文进行转发时使用，NHLFE在ingress和Transit存在，NHLFE包括：Tunnel ID,出接口，下一跳，出标签，标签操作等信息
  - ILM（Incoming Label Map，入标签映射）：用于指导MPLS报文的转发，ILM只存在Transit和Egress，ILM包括：Tunnel ID,入 接口，入标签，标签操作等信息
 - LFIB：标签转发信息表
 - LIB： 标签信息表
 - LSR（Label Switching Router,标间桥换路由器）：支持MPLS的路由器或者交换机，
 - LER（Label Edge Router）:位于MPLS域边缘，链接其他网络的LSR出纳为边沿路由器
 - Core LSR: MPLS域内部的LSR  
 - Ingress LSR:入站LSR，通常想IP报文中压入MPLS头部并生成MPLS报文的LSR
 - Trransit LSR：中转LSR，将MPLS报文进行标签置换操作
 - Egress LSR:出站LSR，将MPLS报文中的MPLS头部移除 
 - FEC(Forwarding Equivalence Class,转发等价类):具有一组某些共性的数据流的集合，通过与MPLS标签对应
 - LSP（Label Switched Path，标签交换路劲）：标签报文穿越MPLS网络达到目的所走的路劲，同一个FEC通过存在用相同过的LSP，所以同一个FEC，LSR总是相同的标签 
  - 单向性，去和回来都需要LSP
- mpls vpn
 - CE ( Customer Edge,用户侧的边缘路由器)，一般链接到PE
 - PE（Provider Edge，运营商的边缘路由器）
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

### ospf DD报文


两台路由器在邻接关系初始化时，用DD报文（Database Description
Packet）来描述自己的LSDB，进行数据库的同步。报文内容包括LSDB中每一条LSA的Header（LSA的Header可以唯一标识一条LSA）。LSA
Header只占一条LSA的整个数据量的一小部分，这样可以减少路由器之间的协议报文流量，对端路由器根据LSA
Header就可以判断出是否已有这条LSA。在两台路由器交换DD报文的过程中，一台为Master，另一台为Slave。由Master规定起始序列号，每发送一个DD报文序列号加1，Slave方使用Master的序列号作为确认。

![ospf-dd](/assets/images/network/ospf-dd.png)

|字段	             |长度|	含义|
|:-------------|:------------------|:------|
|Interface MTU	    |16比特	|在不分片的情况下，此接口最大可发出的IP报文长度。|
|Options	        |8比特	|可选项：E：允许Flood AS-External-LSAs；MC：转发IP组播报文；N/P：处理Type-7 LSAs；DC：处理按需链路。|
|I	                |1比特	|当发送连续多个DD报文时，如果这是第一个DD报文，则置为1，否则置为0。|
|M (More)	        |1比特	|当发送连续多个DD报文时，如果这是最后一个DD报文，则置为0。否则置为1，表示后面还有其他的DD报文。|
|M/S (Master/Slave)	|1比特	|当两台OSPF路由器交换DD报文时，首先需要确定双方的主从关系，Router ID大的一方会成为Master。当值为1时表示发送方为Master。|
|DD sequence number	|32比特	|DD报文序列号。主从双方利用序列号来保证DD报文传输的可靠性和完整性。|
|LSA Headers	    |可变	|该DD报文中所包含的LSA的头部信息。|


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


## IS-IS

- NET（Network Entity Title，网络实体名称）是OSI协议栈中设备的网络层信息，主要用于路由计算，由区域地址（Area ID）、System ID和SEL组成，可以看作是特殊的NSAP（SEL为00的NSAP）。
- NSAP:
![nsap](/assets/images/network/isis-nsap.png)
- DP（Initial Domain Part，初始域部分）相当于IP地址中的主网络号。
 - AFI（Authority and Format Identifier，权限和格式标识符）表示地址分配机构和地址格式,39代表ISO数据国别编码，45代表E.164,49表示本地管理，相当于RFC1918的私有地址；
 - IDI（Initial Domain Identifier，初始域标识符）IDI用来标识域。
- DSP（Domian Specific Part，特定域部分）相当于IP地址中的子网号和主机地址。
 - High Order DSP（高位DSP，HODSP）、System ID（系统ID）和NSEL（NSAP Selector，NSAP选择器）构成。
  - 高位DSP用来将域划分为不同的区域；
  - System ID用来标识OSI设备，区分主机，长度为6字节。
- NSEL类似于TCP或UDP端口号，用于指示服务类型，不同的传输协议对应不同的NSEL，如在IP中，其值为0。
![isis-net](/assets/images/network/isis-net.png)

### 实验

- `display isis peer`
- `display isis lsdb`
- `disp isis route`
- `disp isis interface`
- 进入接口后配置接口的isis的优先级大小为0-127数字越大优先级越高，修改优先级以后这个节点会选举为dis`isis dis-priority 127`
- level1上只有level1 区域自己的路由，可以通过路由渗透把level2的路由引入`import-route isis level-2 into level-1`
- 修改isis的接口类型为p2p,进入接口以后:`isis circuit-type  p2p`


```shell
# 基础配置 
# 配置isis
isis 1 # 默认1
# 配置
# 49.0001 域
# 0000.0000.0001 同一个域内每个交换机不一样
# 00 代表ip
network-entity 49.0001.0000.0000.0001.00
# 设置level
is-level level-1

```
![isis](/assets/images/network/isis-shiyan.png)

## stp

- 修改设备为根桥`stp priority 0`
- R 根端口
- D 指定端口
- A 替代端口

- 只有指定端口才会发送BPDU
- 根桥上所有端口都是指定端口
- 非根桥交换机有且只有一个根端口（去环）
- 根端口是用来接口BPDU的

- BPDU超时时间20s

- DP故障需要等待30s


![bpdu](/assets/images/network/BPDU.png)

## rstp

- R 根端口 
- D 指定端口
- B 备份端口

- 超时时间是三倍的hello time，6秒
- 根端口故障直接切换到备份端口 


- 根保护`stp root-pre`
- 环路保护，在指定的接口下`stp loop-pre`
- stp模式设置`stp mode rstp`
- 配置根桥`stp root primary`
- 配置备份根桥`stp root secondary` 




## mstp

## acl

- rule 5 permit/deny


## route-policy

- isis 引入路由的时候过滤掉指定的路由
- 注意这里的ip ip-prefix需要用permit因为route-policy 用力deny
```shell
isis
import-route ospf 1 route-policy net192
route-policy net192 deny node 10
if-match ip-prefix net192
ip ip-prefix net192 permit 192.168.1.0 24
```
- 引入路由的时候修改优先级,isis优先级是15，这里改为14比isis优先级低

```shell
ospf
import-route isis  route-policy pre
preference ase 14
# 
route-policy pre permit node 10
if-match ip-prefix pre
ip ip-prefix pre permit 192.168.1.0 24
```

## filer-policy

![route-filter](/assets/images/network/route-filter.png)
- 使用filter-policy破环

 
```shell
isis 
filter-policy 2000 import
quit
acl 2000
rule deny source 5.5.5.5 0
# 默认规则，默认允许
rule permit
```


## BGP

- 报文
 - open：建立peer
 - update： 发送路由更新
 - notification： 错误终止对等体
 - keepalive：标志对等简历，维持bgp对等体关系
 - route-refresh: 改变路由策略以后请求对等体重新发送路由信息
- 状态机
 - idel: 开始准备tcp链接
 - connnect: 进入tcp链接，等待完成，如果tcp建立失败则进入active状态
 - active： tcp没有建立成功，反复重试
 - opensent：tcp建立成功，开始发送open包
 - openconffirm: 参数协商
 - established：收到keepalive包，双方协商一致
- 4条通告原则
 1. 今发布路由
 2. 通过EBGP获得的最优路由发布给除路由获取端以外的所有BGP邻居（包括EBGP邻居和IBGP邻居）
 3. 通过IBGP获得的最优路由不会发布给其他的IBGP邻居（水平分割）
 4. BGP与IGP同步（从IBGP邻居学来的路由是否发布给BGP邻居，取决于该路由是否也能通过IGP得知，即BGP和IGP同步）

- 端口179
- 对等，IBGP对等AS号一样，EBGP对等AS号不一样
- 一个路由器只能创建一个bgp
- bgp路由前有`*`表示有效，`>`表示优选，`i`表示对等体（同一个as）发送的



```shell
# 创建bgp AS10
bgp 10
# 创建邻居
peer 10.1.12.2 as-number 20

# 配置静态路由
ip route-static 2.2.2.2 32 10.1.12.2

# 使用lo简历对等体
peer 2.2.2.2 as-number 20
peer 2.2.2.2 connect-interface LoopBack 0

# ebgp 建立peer
peer 1.1.1.1 as-number 10
peer 1.1.1.1 connect-interface l0
peer 1.1.1.1 ebgp-max-hop 2

# 查看路由表，默认是空的
disp bgp routing-table
# 查看详细的
disp bgp routing-table 192.168.1.0
# 路由宣告
network 192.168.1.0 24
# 路由引入
import-route direct/bgp
```