---
title: goroutine
layout: default
parent: go
has_children: false
---


## 1. GMP调度模型

![gmp1](/assets/images/go/gmp1.png)
![gmp2](/assets/images/go/gmp2.png)

- G: 表示goroutine，存储了goroutine的执行stack信息、goroutine状态、goroutine的任务函数以及与所在P的绑定等信息。G对象可以重用。  
- P: 表示逻辑processor，P的数量决定了系统内最大可并行的G的数量（前提：系统的物理cpu核数>=P的数量）。
  - P 还有一个很重要的作用是他拥有的各种 G 对象队列、链表、一些cache和状态  
  - P管理着一组goroutine队列，P里面会存储当前goroutine运行的上下文环境（函数指针，堆栈地址及地址边界），P会对自己管理的goroutine队列做一些调度（比如把占用CPU时间较长的goroutine暂停、运行后续的goroutine等等）当自己的队列消费完了就去全局队列里取，如果全局队列里也消费完了会去其他P的队列里抢任务。 
- M: M表示machine，代表着真正的执行计算资源。
  - M是Go运行时（runtime）对操作系统内核线程的虚拟， M与内核线程一般是一一映射的关系， 一个goroutine最终要放到M上执行的。
  - 在绑定有效的p后，进入schedule循环；而schedule循环的机制大致是从各种队列、p的本地队列中获取G，切换到G的执行栈上并执行G的函数，调用goexit做清理工作并回到m，如此反复。M并不保留G状态，这是G可以跨M调度的基础。 

## 2. 分段栈(segmented stack)

- go通过使用分段栈的方式动态为goroutine分配非连续内存空间通过双向链表关联起来  
- 热分裂问题：如果当前 goroutine 的栈几乎充满，那么任意的函数调用都会触发栈的扩容，当函数返回后又会触发栈的收缩，如果在一个循环中调用函数，栈的分配和释放就会造成巨大的额外开销，这被称为热分裂问题  

## 3. 连续栈

- 调用用 runtime.newstack 在内存空间中分配更大的栈内存空间;  
- 使用 runtime.copystack 将旧栈中的所有内容复制到新的栈中;  
- 将指向旧栈对应变量的指针重新指向新栈;  
- 调用 runtime.stackfree销毁并回收旧栈的内存空间;  


## 3. 并发锁

### 3.1 互斥锁(sync.Mutex)

Go 标准库中提供了 sync.Mutex 互斥锁类型及其两个方法：
Lock 加锁
Unlock 释放锁
通过在代码前调用 Lock 方法，在代码后调用 Unlock 方法来保证一段代码的互斥执行

### 3.2  读写锁(sync.RWMutex)

- 写锁之间是互斥的，存在写锁，其他写锁阻塞。
- 读锁之间不互斥，没有写锁的情况下，读锁是无阻塞的，多个协程可以同时获得读锁。
- 写锁与读锁是互斥的，如果存在读锁，写锁阻塞，如果存在写锁，读锁阻塞。

- Go 标准库中提供了 sync.RWMutex 互斥锁类型及其四个方法：  
> Lock 加写锁  
> Unlock 释放写锁  
> RLock 加读锁  
> RUnlock 释放读锁  

### 3.3 互斥锁有两种状态：正常状态和饥饿状态。

- 正常状态 

> 在正常状态下，所有等待锁的 goroutine 按照FIFO顺序等待。  
> 唤醒的 goroutine 不会直接拥有锁，而是会和新请求锁的 goroutine 竞争锁的拥有。  
> 新请求锁的 goroutine 具有优势：它正在 CPU 上执行，而且可能有好几个，所以刚刚唤醒的 goroutine 有很大可能在锁竞争中失败。  
> 在这种情况下，这个被唤醒的 goroutine 会加入到等待队列的前面。   
> 如果一个等待的 goroutine 超过 1ms 没有获取锁，那么它将会把锁转变为饥饿模式。  

- 饥饿状态

> 在饥饿模式下，锁的所有权将从 unlock 的 goroutine 直接交给交给等待队列中的第一个。
> 新来的 goroutine 将不会尝试去获得锁，即使锁看起来是 unlock 状态, 也不会去尝试自旋操作，而是放在等待队列的尾部。