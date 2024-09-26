---
title: "mpls"
layout: default
parent: Network
has_children: false
---



## 基本信息

- PHP（Penultimate Hop Popping）倒数第二跳，可在倒数第二跳节点处将标签弹出，最后一跳节点直接进行IP转发或者下一层标签转发，减少最后一跳标签交换的负担
    - `label advertise { explicit-null | implicit-null | non-null }`，配置向倒数第二跳分配的标签。
    - 缺省情况下，使用的是implicit-null，表示支持PHP。Egress向倒数第二跳节点分配隐式空标签，值为3
    - 如果配置的是explicit-null，表示不支持PHP。Egress节点向倒数第二跳分配显式空标签，值为0。当需要支持MPLS QoS属性时，可以选用explicit-null。
    - 如果配置的是non-null，表示不支持PHP。Egress向倒数第二跳正常分配标签，即分配的标签值不小于16。

- label advertise { explicit-null | implicit-null | non-null }
    - explicit-null	不支持PHP特性，出节点向倒数第二跳分配显式空标签。	显式空标签的值为0。
    - implicit-null	支持PHP特性，出节点向倒数第二跳分配隐式空标签。	隐式空标签的值为3。
    - non-null	不支持PHP特性，出节点向倒数第二跳正常分配标签。	分配的标签值不小于16。

- **0标签**
    - 表示该标签必须被弹出（即标签被剥掉），且报文的转发必须基于IPv4。如果出节点分配给倒数第二跳节点的标签值为0，则倒数第二跳LSR需要将值为0的标签正常压入报文标签值顶部，转发给最后一跳。最后一跳发现报文携带的标签值为0，则将标签弹出。

- **3标签**
    - 倒数第二跳LSR进行标签交换时，如果发现交换后的标签值为3，则将标签弹出，并将报文发给最后一跳。最后一跳收到该报文直接进行IP转发或下一层标签转发。
## ldp

- ldp是动态mpls标签协议
- LDP使用UDP和TCP两种承载协议，端口号都为646（其中Hello报文基于UDP，其余报文基于TCP）

![message](/assets/images/network/mpls-ldp-message.png)


- 标签发布和管理,**当前设备支持如下组合**：
    - 下游自主方式（DU）＋有序标签控制方式（Ordered）＋自由标签保持方式（Liberal）。
        - 是指对于一个特定的FEC，LSR无须从上游获得标签请求消息即进行标签分配与分发
    - 下游按需方式（DoD）＋有序标签控制方式（Ordered）＋保守标签保持方式（Conservative）。
        - 是指对于一个特定的FEC，LSR获得标签请求消息之后才进行标签分配与分发。
    - 缺省采用的标签发布和管理方式为：下游自主方式（DU）＋有序标签控制方式（Ordered）＋自由标签保持方式（Liberal）。

- 标签分配控制方式:
    - 独立标签分配控制（Independent），是指本地LSR可以自主地分配一个标签绑定到某个FEC，并通告给上游LSR，而无需等待下游的标签
    - 有序标签分配控制（Ordered），指对于LSR上某个FEC的标签映射，只有当该LSR已经具有此FEC下一跳的标签映射消息、或者该LSR就是此FEC的出节点时，该LSR才可以向上游发送此FEC的标签映射


**LDP邻接体**

- 当一台LSR接收到对端LSR发送到Hello消息后，两端之间就建立了LDP邻接体关系

**两种邻接体类型**

本地邻接体：以组播224.0.0.2为目的地址发送Hello消息发现的邻接体

远端邻接体：以单播为目的地址发送Hello消息发现的邻接体（手工指定邻接体）

**LDP会话**

LDP邻接体建立LDP会话，用于在LSR之间交换标签映射、释放会话等信息

**两种会话类型**

本地LDP会话：建立会话的两个LSR之间是直连的本地邻接体关系

远端LDP会话：建立会话的两个LSR之间可以是直连本地邻接关系、也可以是非直连的远端邻接关系

LDP对等体--构建LSP

两个LSR之间存在着LDP会话，可以直接使用LDP来交换标签信息构建LSP，这两个LSR就为LDP对等体

**注意事项**

本地LDP会话可以和远端LDP会话共存

LDP通过邻接体来维护对等体的存在，对等体的类型取决于维护它的邻接体的类型（如果由本地和远端邻接体共同维护，则对等体类型为本远共存对等体）

一个对等体可以由多个邻接体来维护


## mpls TTL处理方式



- **都是Uniform模式则mpls的一个几点也是相当于ip的一个节点ttl保持一致，没过一个节点减1**
- **剩下的模式，中间的mpls不管怎么操作，就看着ip的一个节点，出来的时候ip的ttl减1就行**
- **任何模式在mpls节点的时候ip的ttl是不变的**
- Pipe模式就是mpls的ttl从255开始

- 都是Uniform
- ![ttl-1](/assets/images/network/mpls-ttl-1.png)
    - 都是Uniform模式那就相当于拍平了，每个mpls也是ip的ttl

- 私有的是Pipe
![ttl-2](/assets/images/network/mpls-ttl-2.png)
    - 公网的ttl映射是私有的，私有的是Pipe则公有的也是255，在出的时候共有的映射到私有，ip的ttl直接减1，和mpls的没有关系，中间的所有mpls可以看着ip的ttl减1

- 私有的Uniform，
![ttl-3](/assets/images/network/mpls-ttl-3.png)
    -  私有的Uniform，则私有的与ip的ttl一样，共有的从255 开始，这种情况下中间节点的mpls没有变化，出接口的ip 的ttl减1

- 都是Pipe
![ttl-4](/assets/images/network/mpls-ttl-4.png)
    - mpls都是255 ，在出节点，首先公网标签直接弹出；然后私网标签也直接弹出，IP TTL只在出节点减1。如图MPLS TTL的处理（4）所示。


## mpls vpn
- RT（Route Target）
- RD（route distinguisher）作用就是让BGP可以区分重复的路由前缀,每个VRF（VPN实例）都必须配置至少一个RD

- **RT的两个方向区别**：

    - Export Target：本地PE从直接相连Site学到IPv4路由后，转换为VPN-IPv4路由，并为这些路由设置Export Target属性。Export Target属性作为BGP的扩展团体属性随路由发布。
    - Import Target：PE收到其它PE发布的VPN-IPv4路由时，检查其Export Target属性。当此属性与PE上某个VPN实例的Import Target匹配时，PE就把路由加入到该VPN实例中。

- **RD和RT的区别**：
    - RD是用来区分不同路由的，一个VRF只能由一个RD，且一条VPNv4路由也只有一个RD，但可以关联多个RT