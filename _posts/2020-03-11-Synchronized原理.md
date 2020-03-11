---
title: Synchronized原理
tags: Synchronized原理
sidebar:
  nav: docs-zh
---

### 基本使用

> 原子性：确保线程互斥的访问同步代码；

> 可见性：保证共享变量的修改能够及时可见，其实是通过Java内存模型中的 “对一个变量unlock操作之前，必须要同步到主内存中；如果对一个变量进行lock操作，则将会清空工作内存中此变量的值，在执行引擎使用此变量前，需要重新从主内存中load操作或assign操作初始化变量值” 来保证的；
  
> 有序性：有效解决重排序问题，即 “一个unlock操作先行发生(happen-before)于后面对同一个锁的lock操作”；
  
### 用法

> 当synchronized作用在实例方法时，监视器锁（monitor）便是对象实例（this）；

> 当synchronized作用在静态方法时，监视器锁（monitor）便是对象的Class实例，因为Class数据存在于永久代，因此静态方法锁相当于该类的一个全局锁；
  
> 当synchronized作用在某一个对象实例时，监视器锁（monitor）便是括号括起来的对象实例；
  
### 实现原理

每个对象都是一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象

monitorenter：线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

> 如果monitor的计数器为0，则该线程进入monitor，然后将计数器加1，该线程即为monitor的所有者；

> 如果线程已经占有该monitor，只是重新进入，则进入monitor的计数器加1；
 
> 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的计数器为0，再重新尝试获取monitor的所有权；
  
monitorexit：执行monitorexit的线程必须是objectref所对应的monitor的所有者。指令执行时，monitor的计数器减1，如果减1后计数器为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

> monitorexit指令出现了两次，第1次为同步正常退出释放锁；第2次为发生异步退出释放锁；