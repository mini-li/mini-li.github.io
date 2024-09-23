---
title: "ospf"
layout: default
parent: Network
has_children: false
---

## 1. 基础信息

- DR 选举是非抢占的
    - 如果新加入优先级高的需要reset现有的DR和BDR才会把优先级高的设置为DR（默认为1，0代表放弃），如果优先级相同使用route-id,并且值大的优先
- stub路由器的lsa的度量值为65535
- 命令用来设置进行负载分担的等价路由的最大数量。`maximum load-balancing 1`,如果为1的话就没有负载均衡，最大值是16
    - maximum load-balancing 为1时，如果路由的优先级相同则比较接口ID大的，选择大的作为出接口
- ospf的router id变化以后需要重置进程,在**用户视图**下操作`reset ospf 1 process`
    - 在不同的area交换机的route id可以相同，能建立邻居也能学习到路由
    - 在相同的area里面如果route id相同，如果两个路由器是直连的是不能建立邻居的，如果不是直连的是可以建立邻居但是不能学到相互的路由信息
- ospf的route id是32bit，也就是ipv4的大小，0区域的就是0.0.0.0
- ipv4使用的ospfv2;ipv6使用ospfv3
- ospf时链路状态路由协议
- 路由器收到lsa以后会放入自己的lsdb中，然后分析lsa得到全网的拓扑
- 使用SPF（Shortest Path First）算法
- OSPF路由的优先级为10。当指定ASE(ase表示设置AS-External路由的优先级。)时，缺省优先级为150。
- 224.0.0.5指代在任意网络中所有运行OSPF进程的接口都属于该组，于是接收所有224.0.0.5的组播数据包。
- 224.0.0.6指代一个多路访问网络中DR和BDR的组播接收地址，
- OSPF用IP报文直接封装协议报文，协议号为89
- OSPF的3张表：邻居表`display ospf peer`、LSDB表`display ospf lsdb`、OSPF路由表`disp ospf routing`  
- OSPF的4种网络类型: 广播类型（Broadcast 或 MA）也是默认类型、P2P、NBMA、P2MP
    - 广播：
        - Hello、LSU、LSACK通过组播发送，DD与LSR通过单播发送。
        - 默认Hello 10秒，Dead 4*10=40秒（响应超时是4倍时间）。
    - NBMA:
        - 默认Hello 30秒，Dead 30*4=120秒（响应超时是4倍时间）。
    - p2p:
        - 默认链路为串口类型PPP、HDLC时，该链路的OSPF网络类型为P2P类型。
        - 所有发送的OSPF报文（Hello，DD，LSR，LSU，LSACK）都通过组播
        - 默认Hello10秒，Dead40秒。
- OSPF分为5种报文：
    - Hello报文、组播报文，协议号89，使用224.0.0.5，默认状态为Down,如果从邻居收到的报文中不包含自己的router id则状态变为init,如果包含则状态为2-way
    - DD报文、主从协商
    - LSR报文、
    - LSU报文
    - LSAck报文。
- OSPF的7种状态：
    - Down     ：没有收到Hello包                         
    - Attempt （一般不存在）
    - Init     ：收到邻居发来的Hello包，但是收到Hello包中的邻居字段没有自己       
    - 2-Way    ：收到邻居发来的Hello包的邻居列表中有自己，建立邻居关系        
    - Exstart  ：  
        发送DD报文（此处DD报文不包含LSA头部信息）  
        **作用**  
        决定主从关系（Router-ID大的为主路由器，小的为从路由器）  
        确定序列号保证报文交互的可靠性  
        比较MTU（可选，缺省不比较）  
        **MTU对邻居建立的影响**  
        1、主的MTU值比从设备的MTU值小； 从设备会停留在Exchange状态，主设备停留在Exstart状态
        2、主的MTU值比从设备的MTU值大； 两端都会停留在Exstart状态
        3、两端有一段未开启MTU值检测，则两端可以建立邻居  
    - Exchange  
        通过交换DD报文，交换LSA头部信息  
        **具体的交互流程**  
        上述Exstart状态决定出主从关系后，即从设备此时收到了主设备发来的空的DD报文  
        从设备使用主的序列号发送DD报文来回应主（此时DD报文包含LSA头部信息）  
        主也通过DD报文发送自己的LSA头部信息，并将序列号加1  
        从又使用主的序列号回应主；一直循环，直到主与从的M位都不置位（或者说只要有一侧有未传递的LSA头，就会一直循坏）    
    - Loading   
        发送LSR，LSU，LSACK
        通过上述获得的LSA头部信息，来确定自己需要哪些LSA  
        **具体实现方式**  
        通过发送LSR请求、发送自己的LSA完整信息（LSU）给对方、发送LSACK确认信息来完成LSDB同步  
    - Full: LSDB同步完成，建立邻接关系

- 不同网络DR和BDR选举
![ospf-DR选举](/assets/images/network/ospf-DR选举.png)



## 2. 名词解释

- OSPF: Open Shortest Path First: 开放式最短路径优先
    - DR （Designated Router）：多接入网络中的指定路由器
    - BDR： 备份指定路由
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

- LSA三元组
    - TYpe (上面的7种类型)
    - LinkStateID
        - Type1中Link State ID: 生成这条LSA的路由器的Router ID。
        - Type2中Link State ID: 描述网段上DR的端口IP地址。
        - Type3中Link State ID: 描述区域内网段。
        - Type4中Link State ID: ASBR的Route ID 。
        - Type5中Link State ID: 外部路由的网段。
        - Type7中Link State ID: 外部路由的网段。

    - AdvRouter （宣告的路由器）

  

## 3.  报文交互过程


![ospf-报文交互](/assets/images/network/ospf-报文交互过程.png)


## 4. 报文，[参考](https://support.huawei.com/enterprise/zh/doc/EDOC1100278276/aa8766d8)

- 报文头,五种报文使用相同过的报文头格式

![ospf报文头](/assets/images/network/ospf报文头.png)

- 报文头格式解释

![格式说明](/assets/images/network/ospf报文头格式说明.png)


### 4.1 hello报文

- 最常用的一种报文，其作用为建立和维护邻接关系，周期性的在使能了OSPF的接口上发送。报文内容包括一些定时器的数值、DR、BDR以及已知的邻居
- Options: 8bit
    - E：允许泛洪AS-External-LSAs
    - MC：转发IP组播报文
    - N/P：处理Type-7 LSAs (NSSA区域)
    - DC：处理按需链路
    - DN：用来避免在MPLS VPN中出现环路。当PE向CE发送3类、5类和7类LSA时需要设置DN位，其他PE从CE接收到该LSA时，不能在它的OSPF路由计算中使用该LSA。
    - O：用来定义始发路由器是否支持Opaque LSA（9类、10类和11类）。
- router priority: DR优先级，默认为1 如果是0则路由器不参与DR或BDR的选举
- Designated Rouer: DR的接口地址（DR一定存在，BDR不一定存在）
- Back Designated Router: BDR的地址
![hello-head](/assets/images/network/ospf-hello-head.png)

### 4.2 DD报文

- Interface MTU: 在不分片的情况下，此接口最大可发出的IP报文长度，单位为字节。
- I (init)： 初始位，当发送连续多个DD报文时，如果这是第一个DD报文，则置为1，否则置为0。
- M (More):更多位，当发送连续多个DD报文时，如果这是最后一个DD报文，则置为0。否则置为1，表示后面还有其他的DD报文。
- MS (Master/Slave): 主从位，当两台OSPF路由器交换DD报文时，首先需要确定双方的主从关系，Router ID大的一方会成为Master。当值为1时表示发送方为Master。
- DD sequence number: DD报文序列号。主从双方利用序列号来保证DD报文传输的可靠性和完整性。


```txt
0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +---------------+---------------+-------------------------------+
    |  Version = 2  |       2       |         Packet length         |
    +---------------+---------------+-------------------------------+
    |                          Router ID                            |
    +---------------------------------------------------------------+
    |                           Area ID                             |
    +-------------------------------+-------------------------------+
    |           Checksum            |             AuType            |
    +-------------------------------+-------------------------------+
    |                       Authentication                          |
    +---------------------------------------------------------------+
    |                       Authentication                          |
    +-------------------------------+---------------+-+-+-+-+-+-+-+-+
    |         Interface MTU         |    Options    |0|0|0|0|0|I|M|MS
    +-------------------------------+---------------+-+-+-+-+-+-+-+-+
    |                     DD sequence number                        |
    +---------------------------------------------------------------+
    |                                                               |
    +-                                                             -+
    |                                                               |
    +-                      An LSA Header                          -+
    |                                                               |
    +-                                                             -+
    |                                                               |
    +-                                                             -+
    |                                                               |
    +---------------------------------------------------------------+
```




