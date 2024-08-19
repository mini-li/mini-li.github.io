---
title: type
layout: default
parent: go
has_children: false
---


## 1. 数据类型

- 值类型： int系列、float系列、bool、string、数组和结构体  
- 引用类型：指针、slice切片、管道channel、接口interface、map、函数  

{: .important }
go语言中所有的传参都是值传递，新拷贝了一个副本。

## 2. 切片

![slice](/assets/images/go/slice.png)

- 切片是长度可变、容量固定的相同的元素序列
- Go语言的切片本质是一个数组，容量固定是因为数组的长度是固定的，切片的容量即隐藏数组的长度
- 切片的创建
  - make ( []Type ,length, capacity )
  - make ( []Type, length)
  - []Type{}
  - []Type{value1 , value2 , ... , valueN }
  - shuzu[1:]
- 数组指针与指针数组
  - 针数组是一系列指针组成的数组
  - 数组指针是是数组的指针

{: .warning }
如果是用一个数组创建切片在切片没有触发扩容操作时修改数组会修改切片的值  

## 3. map

- map本质上是哈希表（hash table）哈希表是一种使用哈希函数组织数据,以支持快速插入和搜索的数据结构
- 实现哈希表的两个关键是哈希函数和解决哈希冲突。

完美的hash函数是不通的输入key得到不同的Value

![map](/assets/images/go/map.png)

实际可能是

![map1](/assets/images/go/map1.png)

go里面使用的是桶加链表的形式来解决冲突

- **go中map的结构**

![map struct](/assets/images/go/map2.png)

解决hash冲突的常见两种方案开放寻址法和拉链法

### 3.1 开放寻址法

开放寻址法是一种在哈希表中解决哈希碰撞的方法，这种方法的核心思想是依次探测和比较数组中的元素以判断目标键值对是否存在于哈希表中，如果我们使用开放寻址法来实现哈希表，那么实现哈希表底层的数据结构就是数组

### 3.2 拉链法

大多数的编程语言都用拉链法实现哈希表， 拉链法通常使用数组和链表作为底层数据结构。  

哈希值使用数组将哈希值HashValue相同的Key对应的Value通过链表数组进行维护  