---
title: "stp"
layout: default
parent: Network
has_children: false
---


## 基本信息
通过在交换机之间传递BPDU来选举根交换机，以及确定每个交换机端口的角色和状态
stp分三个版本，stp，rstp，mstp

- BPDU分为配置BPDU和拓扑BPDU

备份端口是收到了自己指定端口发出的dpdu
替代端口是根端口的备份