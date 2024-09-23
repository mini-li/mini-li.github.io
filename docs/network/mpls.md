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