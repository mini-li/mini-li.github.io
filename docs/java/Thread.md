---
title: Thread
layout: default
parent: java
has_children: false
---

## 1. Thread

- Java 线程核心就是Thread类
- 或者实现run方法

```java
public class App {
    public static void main(String[] args) {
        // 创建并启动线程
        MyThread thread = new MyThread();
        thread.start();
    }

    static class MyThread extends Thread {
        @Override
        public void run() {
            // 线程执行的代码
            System.out.println("Thread is running");
        }
    }
}
```
### 1.1 join

- 调用某线程的该方法，将当前线程和该线程合并，即等待该线程结束，再恢复当前线程的运行
```java
public class App {
    public static void main(String[] args) {
        System.out.println("main  start ...");
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                for(int i = 0 ; i < 10; i++){
                    System.out.println(Thread.currentThread().getName() + " 子线程执行了...");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
 
            }
        });
        t1.start();
        System.out.println(t1.isAlive());
        try {
            t1.join(); // 线程的合并，和主线程合并  相当于我们直接调用了run方法
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(t1.isAlive());
        System.out.println("main end ...");
    }
}
```

### 1.2 yield

- 让出CPU，当前线程进入就绪状态

```java
public class App extends Thread{
 
    public App(String threadName){
        super(threadName);
    }
 
    /**
     * yield方法  礼让
     *
     * @param args
     */
    public static void main(String[] args) {
        App f1 = new App("A1");
        App f2 = new App("A2");
        App f3 = new App("A3");
 
        f1.start();
        f2.start();
        f3.start();
    }
 
    @Override
    public void run() {
        for(int i = 0 ; i < 100; i ++){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
 
            if(i%10 == 0 && i != 0){
                System.out.println(Thread.currentThread().getName()+" 礼让：" + i);
                Thread.currentThread().yield(); // 让出CPU
            }else{
                System.out.println(this.getName() + ":" + i);
            }
        }
    }

}
```

### 1.3 interrupt

- 如果被打断线程正在 sleep，wait，join 会导致被打断的线程抛出 InterruptedException，并清除 打断标记；  
- 如果打断的正在运行的线程，则会设置 打断标记；
- park 的线程被打断，也会设置 打断标记

```java
public class App {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            while (true) {
                System.out.println("I'm running!");
                boolean interrupted = Thread.currentThread().isInterrupted();
                if (interrupted) {
                    System.out.println("我来处理中断请求了");
                    break;
                }
            }
        }, "t1");

        t1.start();
        try{
            Thread.sleep(1);
        }catch(Exception e){
            System.out.println(e);
        }
        
        t1.interrupt();
        System.out.println("中断状态：" + t1.isInterrupted());
    }
}
```

## 2. callable 和 FutureTask 获取线程结果

```java

public class App implements Callable<String> {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> futureTask = new FutureTask(new App());
        Thread thread = new Thread(futureTask);
        thread.start();
        String value = futureTask.get();
        System.out.println(value);
    }
    
    @Override
    public String call() throws Exception {
        System.out.println("TestCallable start....");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
 
        System.out.println("TestCallable end");
 
        return "call result";
    }
}
```

## 3. ThreadLocal


- get和set操作都是获取到当前线程的ThreadLocalMap具体操作都会操作到当前线程的

{: .warning }
当在线程池中使用ThreadLocal时一定要remove不然可能会造成内存泄漏


```java
import java.util.concurrent.*;
import java.util.Random;
public class App {
    // 创建一个ThreadLocal对象，用于存储每个线程独立的Integer值，初始值为0
    private static ThreadLocal<Integer> threadLocalValue = ThreadLocal.withInitial(() -> new Random().nextInt());
 
    public static void main(String[] args) {
        // 定义一个Runnable任务，模拟每个线程对ThreadLocal变量的操作
        Runnable task = () -> {
            // 从ThreadLocal中获取当前线程的值，初始时该值为0
            int value = threadLocalValue.get();
            // 对该值进行自增操作
            value++;
            
            try {
                 // 将自增后的值重新存入ThreadLocal中
                threadLocalValue.set(value);
                // 输出当前线程的名称和ThreadLocal中的值
                System.out.println(Thread.currentThread().getName() + ": " + threadLocalValue.get());
            } finally {
                threadLocal.remove();
            }
        };
 
        // 创建两个线程，执行相同的任务
        Thread thread1 = new Thread(task, "Thread 1");
        Thread thread2 = new Thread(task, "Thread 2");
 
        // 启动两个线程
        thread1.start();
        thread2.start();
    }
}
```

## 4. sychronized 和 ReentrantLock的区别

| sychronized        | ReentrantLock          | 
|:-------------|:------------------|
| 自动上锁和解锁           | 手动上锁和解锁 |
| 不可获取当前线程是否上锁 | 可获取当前线程是否上锁（isHeldByCurrentThread）   |
| 非公平锁           | 公平锁或非公平锁(构造函数指定true则公平)      |
| 不可中断           | 可中断: tryLock |
| 锁的是对象，锁信息保存在对象头           | int类型的state标识来标识锁的状态 |
| 锁升级过程 （无->偏向->轻-重）          | 没有锁升级过程 |



