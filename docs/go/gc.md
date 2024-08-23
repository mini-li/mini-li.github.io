---
title: gc
layout: default
parent: go
has_children: false
---


## 1. gc方式

- 在 Go 1.5 版本之前，Go 语言的垃圾回收是通过“stop-the-world”方式实现的，即在进行垃圾回收时，所有 Goroutine 都会被暂停。这意味着在垃圾回收过程中，程序的执行会被中断。  
- 从 Go 1.5 版本开始，Go 引入了并发的垃圾回收（concurrent garbage collection）机制，也称为并发标记清除（concurrent mark and sweep）。这种机制允许垃圾回收与程序的执行并发进行，减少了停顿时间，提高了程序的响应性能。

## 2. gc触发条件和策略

### 2.1 定时触发

> Go 运行时系统会根据一定的时间间隔定期触发垃圾回收。时间间隔根据程序的内存使用情况和性能需求进行自适应调整。

- 可以使用 time 包来实现定时触发的功能,开启`GODEBUG=gctrace=1`可以看到gc的信息

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // 创建一个定时器，每隔1秒触发一次
    timer := time.NewTicker(time.Second)

    // 启动一个 Goroutine 来执行任务
    go func() {
        for {
            select {
            case <-timer.C:
                // 在定时触发时执行任务
                fmt.Println("任务执行")
            }
        }
    }()

    // 等待一段时间，防止程序立即退出
    time.Sleep(5 * time.Second)

    // 停止定时器
    timer.Stop()

    // 输出完成信息
    fmt.Println("任务结束")
}
```

### 2.2 内存分配触发

- 当程序申请的内存超过一定阈值时，Go 运行时会触发垃圾回收，以防止过度使用内存。
- 使用 runtime 包中的 MemStats 结构体和 ReadMemStats 函数获取垃圾回收的状态信息

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    var stats runtime.MemStats
    runtime.ReadMemStats(&stats)

    fmt.Println("Allocated memory:", stats.Alloc)
    fmt.Println("Total memory allocated and not yet freed:", stats.TotalAlloc)
    fmt.Println("Memory allocated from system:", stats.Sys)
    fmt.Println("Number of garbage collections:", stats.NumGC)
    fmt.Println("Last garbage collection pause duration:", stats.PauseNs[(stats.NumGC+255)%256])
    fmt.Println("Last garbage collection pause end time:", stats.PauseEnd[(stats.NumGC+255)%256])
}
```

### 2.3 栈伸缩触发

- 我们无需显式地触发栈伸缩操作，Go 运行时会根据需要自动处理  
- 以下是一个简单的示例代码，演示了栈伸缩的效果：

```go
package main

import (
    "fmt"
    "runtime"
    "sync"
)

func printGoroutineInfo() {
    var stats runtime.StackRecord
    runtime.GoroutineProfile(&stats)

    fmt.Printf("Stack Size: %d bytes\n", stats.Stack0.Size())
    fmt.Printf("Stack In Use: %d bytes\n", stats.Stack0.Inuse())
}

func worker(wg *sync.WaitGroup) {
    defer wg.Done()

    // 打印 Goroutine 信息
    printGoroutineInfo()

    // 执行一些耗时操作
    recursiveCall(1)
}

func recursiveCall(depth int) {
    if depth == 10000 {
        return
    }

    recursiveCall(depth + 1)
}

func main() {
    var wg sync.WaitGroup
    wg.Add(1)

    // 创建一个 Goroutine
    go worker(&wg)

    wg.Wait()
}
```

## 3. 三色标记法

根据可达性从root开始遍历访问，在遍历过程中按照是否检查过对对象标记为三种颜色
- 白色： 该对象没有被标记过
- 灰色： 该对象被标记过，但是对象下的属性没有被完全标记
- 黑色： 该对象被标记过，且对象下的属性也被标记过

### 3.1  stop the world
通过停止程序来标记垃圾这个时候是最准确的因为这个时候不会制造新的垃圾，但是这个时候用户程序也被停止了这个是不被接受的

### 3.2 异步标记

异步标记不会停止用户的程序，但是又引入了新的问题：  

**浮动垃圾**：因为是异步问题这个时候一边标记一边程序又在支持垃圾，但是这个问题不大，gc在下一次执行的时候也会标记到  

**对象漏标记**：在gc一边标记的时候一边这个引用又变了，把垃圾和对象标记错了，这个时候造成对象会回收了，有两种处理方案  
    - Incremen Upate  
    - STAB  

## 4. OOM

### 4.1 造成OOM的可能原因
- 申请的内存太多  
 - 可以考虑优化申请的内存  
 - 提前设置`HeapDumpOnOutOfMemoryError -XX:HeapDumpPath`在发生OOM时候会留下dump文件
- 内存资源没有释放  
 - 比如链接没有释放，可以考虑连接池  
 - 系统运行中导出dump文件`jmap -dump:format=b,file=xx.hprof $pid` **注意这个命令会触发gc的stop the world**
- 本身资源不够用  
 - jmap、jstack 查看具体的情况
