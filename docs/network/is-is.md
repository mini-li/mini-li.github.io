---
title: "is-is"
layout: default
parent: Network
has_children: false
---



## 基础信息


- `default-route-advertise`命令用来配置运行IS-IS协议的设备生成缺省路由。
    - `default-route-advertise always level-1`指定发布的缺省路由级别为Level-1。如果不指定级别，则默认为生成Level-2级别的缺省路由。
- igp
- 链路状态，不是基于tcp/ip的
- 优先级15
- 工作在2曾，只有组播的mac地址 level1:`0180-c200-0014`,level2:`0180-c200-0015`,类似ospf的三层地址`224.0.0.5/6`
- isis路由开销默认为narrow模式开销默认值为10范围为1~63
- isis的lsp默认刷新时间是900秒
- isis的set-overload 重启以后默认推出时间是600秒

- Holding Time:是hello 时间的3倍，默认hello 间隔是10s

- dis（Designated Intermediate System），在广播网络中才存在，在建立邻居以后等两个hello 报文间隔，再进行DIS选举
    - 优先级0-127，默认64，且该字段越大越优先。当该字段值相等时，则比较MAC地址，越大越优先。
    - 区分level1 和level2
    - dis是抢占式的，如果新路由器加入并且符合成为新的dis，这马上成为新的dis，会引起一组新的lsp泛红，而在ospf中，不会
    - 周期发送csnp，同步lsdb
    - 时间间隔是两倍hello间隔
    - 所有链路之间建立邻居关系
    - hello PDU是普通路由器的1/3
- 邻居关系的建立：
    - 点点网络是两次握手/三次握手，华为默认是三次
    - 广播网络是三次握手
- OL(overload),系统重启以后维护过载时间默认是600s

- **NSAP**

![nsap](/assets/images/network/isis-NSAP.png)

- **NET**
    - 一个路由器可以配置三个net但是SystemID需要相同
![isis-net](/assets/images/network/isis-net01.png)

- 骨干区域：l2和l1-2组成的连续区域
- isis的路由器划分区域的时候，整个路由器都必须属于某个区域，与ospf不同，不能通过接口接入不同的区域
- 没有邻居和邻接区分，ospf有区分

- level2 是骨干路由，维护了所有路由
- 支持的网络类型
    - 广播 boardcast，需要选dis，可以修改为p2p，`isis circuit-type p2p`
    - 点到点 p2p ，不需要选dis, 不可以反向修改为广播

- 三张表
    - peer `disp isis peer`
    - 数据库`disp isis lsdb` 也是基于spf算法
    - 路由表`disp isis route`

- 默认开销为10,开销类型为narrow
    - 接口开销，优先级最高
    - 全局开销，为所有接口设置开销，优先级中等
    - 自动计算开销，根据带宽计算，优先级最低
    - 开销类型
        - narrow (默认)，范围1~63
        - wide，最大值16777215
## 报文

- 头部PDU，可分为4种类型(ospf是五种)：
    - Hello (IIH,isis hello),建立和维护邻居管理，在广播level1中使用的level-1 LAN IIH，在广播网L2使用leve2-LAN-IIH；在点到点使用P2P IIH
    - LSP （Link State PDUs）  分为level1 lsp、level2 lsp，Level-1-2 IS-IS则可传送以上两种LSP。 Level-1设备对于收到ATT位置为1的LSP报文，会生成一条目的地为发送该LSP的Level-1-2设备地址的缺省路由。[报文参考](https://support.huawei.com/enterprise/zh/doc/EDOC1100033741/5825b487)
        -  `attached-bit advertise never` 不ATT置位  
        -  `attached-bit advertise always` ATT置位  
        - 这里需要注意的是，level1与level2在不同的区域才会有这种情况
        - LSP刷新周期的缺省值为900秒。
    - SNP 描述全部的或者部分链路数据中的LSP来同步个各LSDB，类似于ospf的DD
        - CSNP（Complete Sequence Number PDU,全部序列报文），区分level 1和level 2
        - PSNP（Partial Sequence Number PDU， 部分序列报文），区分level 1和level 2

![isis-type](/assets/images/network/isis-报文类型.png)

- IS-IS PDU可以分为两个部分，报文头和变长字段部分。其中头部又可分为通用头部和专用头部
- **通用报头格式**
```TXT
   +----------------------------------------------+------------
   |  Intradomain Routing Protocol Discriminator  |      |
   +----------------------------------------------+      |
   |               Length Indicator               |      |
   +----------------------------------------------+      |
   |        Version/Protocol ID Extension         |      |
   +----------------------------------------------+      |
   |              SYSTEMID Length                 |     PDU
   +----------------------------------------------+    Common
   | R | R | R |          PDU Type                |    Header
   +----------------------------------------------+      |
   |                    Version                   |      |
   +----------------------------------------------+      |
   |                   Reserved                   |      |
   +----------------------------------------------+      |
   |             Maximum Area Addresses           |      |
   +----------------------------------------------+------------
   |              PDU Specific Header             |
   +----------------------------------------------+
   |         Variable Length Fields (CLV/TLC)     |
   +----------------------------------------------+
```
- 各字段含义

| 字段      | 长度 |    含义    |
| ----------- | ----------- | ----------- |
|Intradomain Routing Protocol Discriminator     |1字节|   域内路由选择协议鉴别符，设置为0x83。|
|Length Indicator                               |1字节|   PDU头部的长度（包括通用头部和专用头部），以字节为单位。|
|Version/Protocol ID Extension                   |1字节|   版本/协议标识扩展，设置为1（0x01）|
|SYSTEMID Length                                 |1字节|   NSAP地址或NET中System ID区域的长度。值为0时，表示System ID区域的长度为6字节。值为255时，表示System ID区域为空（即长度0）。|
|R                                               |1比特|   保留(Reserved)位，设置为0。|
|PDU Type                                        |5比特 |  PDU的类型。IS-IS PDU共有9种类型，详细信息请参考表1-35。|
|Version                                         |1字节 |  IS-IS版本号，设置为1（0x01）。|
|Maximum Area Addresses                          |1字节 |  支持的最大区域个数。设置为1～254的整数，表示该IS-IS进程实际所允许的最大区域地址数；设置为0，表示该IS-IS进程最大只支持3个区地址数。|
|PDU Specific Header                             |变长|    专用头部，根据PDU类型不同而有所差别。|
|Variable Length Fields                |变长 |   由多个CLV（Code-Length-Value）三元组组成。CLV也称为TLV（Type-Length-Value），其格式如图1-65所示。不同PDU类型所包含的CLV是不同的，如表1-36所示，其中，Code值从1到10的CLV在ISO 10589中定义（有2类未在上表中列出），其他几种CLV在RFC 1195中定义。|

- **TLV**
![isis-tlv](/assets/images/network/isis-tlv.png)
- **229** TLV支持多拓扑

![isis-tlv-129](/assets/images/network/isis-tlv-129.png)
- **232/236**
![isis-tlv-129](/assets/images/network/isis-tlv-236.png)