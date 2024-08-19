---
title: pointer
layout: default
parent: go
has_children: false
---

## 1. 指针地址和变量空间

Go语言保留了指针, 但是与C语言指针有所不同. 主要体现在:

- 默认值: nil.  
- 操作符 & 取变量地址, * 通过指针访问目标对象.  
- 不支持指针运算, 不支持 -> 运算符, 直接用 . 访问目标成员.  

指针就是地址, 指针变量就是存储地址的变量：

```go
package main

import "fmt"

func main(){ 
    var x int = 99
    var p *int = &x

    fmt.Println(p)
}
```

## 2. 栈帧的内存布局

32位内存地址布局：  

![pointer](/assets/images/go/pointer.png)

- 其中, 数据区保存的是初始化后的数据 
- 一般 `make()` 或者 `new()` 出来的都存储在堆区  
- 当函数调用时, 产生栈帧; 函数调用结束, 释放栈帧.

### 2.1 那么栈帧用来存放什么?

- 局部变量  
- 形参  
- 内存字段描述值  

首先运行 main(), 这时就产生了一个栈帧.

![pointer1](/assets/images/go/pointer1.png)

## 3. 空指针与野指针

空指针: 未被初始化的指针,如`var p *int`  
野指针: 被一片无效的地址空间初始化，如 `var p *int = 0xc00000a0c8` 这里只是一个比如，在go里是不能直接给指针赋值一个数字
