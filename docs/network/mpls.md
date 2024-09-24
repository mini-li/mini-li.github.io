---
title: "mpls"
layout: default
parent: Network
has_children: false
---



## 基本信息
- label advertise { explicit-null | implicit-null | non-null }
    - explicit-null	不支持PHP特性，出节点向倒数第二跳分配显式空标签。	显式空标签的值为0。
    - implicit-null	支持PHP特性，出节点向倒数第二跳分配隐式空标签。	隐式空标签的值为3。
    - non-null	不支持PHP特性，出节点向倒数第二跳正常分配标签。	分配的标签值不小于16。


## ldp

- ldp是动态mpls标签协议
- LDP使用UDP和TCP两种承载协议，端口号都为646（其中Hello报文基于UDP，其余报文基于TCP）

![message](/assets/images/network/mpls-ldp-message.png)


**LDP邻接体**

- 当一台LSR接收到对端LSR发送到Hello消息后，两端之间就建立了LDP邻接体关系

**两种邻接体类型**

本地邻接体：以组播224.0.0.2为目的地址发送Hello消息发现的邻接体

远端邻接体：以单播为目的地址发送Hello消息发现的邻接体（手工指定邻接体）

LDP会话

LDP邻接体建立LDP会话，用于在LSR之间交换标签映射、释放会话等信息

两种会话类型

本地LDP会话：建立会话的两个LSR之间是直连的本地邻接体关系

远端LDP会话：建立会话的两个LSR之间可以是直连本地邻接关系、也可以是非直连的远端邻接关系

LDP对等体--构建LSP

两个LSR之间存在着LDP会话，可以直接使用LDP来交换标签信息构建LSP，这两个LSR就为LDP对等体

注意事项

本地LDP会话可以和远端LDP会话共存

LDP通过邻接体来维护对等体的存在，对等体的类型取决于维护它的邻接体的类型（如果由本地和远端邻接体共同维护，则对等体类型为本远共存对等体）

一个对等体可以由多个邻接体来维护
