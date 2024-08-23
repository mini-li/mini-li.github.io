---
title: ClassLoader
layout: default
parent: java
has_children: false
---

## 1. 类加载器的种类
- Bootstrap ClassLoader 启动类加载器  
- Extention ClassLoader 标准扩展类加载器    
- Application ClassLoader 应用类加载器  
- User ClassLoader 用户自定义类加载器  

{: .important }
一般认为上一层加载器是下一层加载器的父加载器，那么，除了BootstrapClassLoader之外，所有的加载器都是有父加载器的。



## 2. 双亲委派机制

因为类加载器之间有严格的层次关系，那么也就使得Java类也随之具备了层次关系（或者说这种层次关系是优先级）。    

委派的原理：
比如一个用户自定义的com.hollis.ClassHollis类，他会被一直委托到Bootstrap ClassLoader，但是因为Bootstrap ClassLoader不负责加载该类，那么会在由Extention ClassLoader尝试加载，而Extention ClassLoader也不负责这个类的加载，最终才会被Application ClassLoader加载。
委派的好处：
- 通过委派的方式，可以避免类的重复加载，当父加载器已经加载过某一个类时，子加载器就不会再重新加载这个类。
- 通过双亲委派的方式，还保证了安全性。