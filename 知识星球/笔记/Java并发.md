---
title: Java并发
tags: Java并发
sidebar:
  nav: docs-zh
---

# Java并发

## 进程与线程

### 1、线程、进程、程序

**进程**是程序的实例，是程序（浏览器）的一次执行实例，进程可以分为一到多个线程。

线程作为最小调度单位，进程作为资源分配的最小单位。

### 2、并发、并行

并发（concurrent）是同一时间应对多件事情的能力

并行（parallel）是同一时间动手做多件事情的能力

**单核 cpu 下，多线程不能实际提高程序运行效率，线程实际还是串行执行的，只是为了能够在不同的任务之间切换，不同线程轮流使用 cpu**

## Java线程

### 1、创建线程的四种方式

1）继承Thread类创建线程（定义Thread类子类，重写run()方法，调用start()方法启动线程）

2）实现Runnable接口创建线程（定义Runnable接口实现类，重写run()方法，将该类实例作为target创建Thread对象，调用start()方法启动线程）

3）使用Callable和Future创建线程（创建Callable接口实现类，实现call()方法，使用FutureTask类来包装Callable对象，该FutureTask对象封装了Callable对象的call()方法的返回值，使用FutureTask对象作为Thread对象的target创建并启动线程（因为FutureTask实现了Runnable接口），调用FutureTask对象的get()方法来获得子线程执行结束后的返回值）

Callable接口提供了一个call（）方法作为线程执行体，call()方法可以有返回值，可以声明抛出异常，Java5提供了Future接口来代表Callable接口里call()方法的返回值，并且为Future接口提供了一个实现类FutureTask，这个实现类既实现了Future接口，还实现了Runnable接口，因此可以作为Thread类的target。

4）使用线程池例如用Executor框架

### 2、常见方法

#### Thread类

**start()** 让线程进入就绪状态，当得到CPU时间片时才会运行

**run()** 线程启动后调用的方法（直接调用不会创建线程执行，Runnable方式创建，线程启动后调用Runnable的run()方法，继承Thread则是调用覆盖的run()方法）

**join()** 当前线程里调用其它线程t的join方法，当前线程进入WAITING/TIMED_WAITING状态，当前线程不会释放已经持有的对象锁

**isInterrupted()** 判断是否被打断，不会清除打断标记

**interrupt()** 打断线程（如果被打断线程正在 sleep，wait，join 会导致被打断的线程抛出 InterruptedException，并清除打断标记 ；如果打断的正在运行的线程，则会设置打断标记 ；park 的线程被打断，也会设置 打断标记）

**interrupted()** （static）判断当前线程是否被打断会清除打断标记

**sleep(long n)** 线程进入阻塞状态，不释放锁资源（static）让当前执行的线程休眠n毫秒，并让出cpu的时间片给其它线程

**yield()** 线程进入就绪状态，不释放锁资源（static）提示线程调度器让出当前线程对CPU的使用

**stop()** 停止线程运行 比较暴力，已过时

#### Object类

**wait()** 当前线程释放对象锁，进入等待队列 使用是需要对象锁，而LockSupport.park()不需要

**notify()** 唤醒在此对象监视器上等待的单个线程，选择是任意性的。notifyAll()唤醒在此对象监视器上等待的所有线程。

### 守护线程

为用户线程服务，只要非守护线程运行结束，即使守护线程没有执行完，也会强制结束。

可以通过Thread.setDaemon(true)设置线程为守护线程（在start()方法之前）

### 线程状态


**初始状态NEW** 实现Runnable接口和继承Thread可以得到一个线程类，new一个实例出来

**就绪状态（运行中状态）** 调用start()方法进入就绪状态，获得CPU时间片进入运行中状态，用完时间片、调用当前线程yield()或获取到锁的线程进入就绪状态

**阻塞状态** 线程阻塞在进入synchronized或者lock时的状态

**等待状态** 此状态的线程不会被分配CPU执行时间，它们要等待被显式地唤醒

**超时等待状态** 处于这种状态的线程不会被分配CPU执行时间，不过无须无限期等待被其他线程显示地唤醒，在达到一定时间后它们会自动唤醒。

**终止状态** 当线程的run()方法完成时，或者主线程的main()方法完成时，就认为它终止

## Synchronized

### 1、介绍

Synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性。加锁解锁由JVM实现。

### 2、用法

> 用于普通同步方法，锁是当前实例对象

> 用于静态同步方法，锁是当前类的class对象

> 用于同步方法块，锁是括号里面的对象

### 3、原理

**Java对象头** synchronized用的锁存在对象头中，对象头包含Mark Word（标记字段，用于存储对象自身运行时数据）、Klass Pointer（类型指针，对象指向它的类元数据指针）

**Monitor监视器** 每个对象头都可以关联一个Monitor对象，当用Synchronized给对象上锁后，对象头的 MarkWord 中的LockWord指向Monitor的起始地址

刚开始 Monitor中Owner为null，当第一个线程执行synchronized(obj)就会将Monitor的所有者Owner置为自己，Monitor中只能有一个 Owner，如果其它线程也来执行 synchronized(obj)，就会进入阻塞队列 BLOCKED，当第一个线程执行完同步代码块的内容，然后唤醒阻塞队列中等待的线程来竞争锁，竞争的时是非公平的

**实现**

- 同步代码块是使用 monitorenter 和 monitorexit 指令实现的；

- 同步方法（在这看不出来需要看JVM底层实现）依靠的是方法修饰符上的ACC_SYNCHRONIZED 实现。

### 4、轻量级锁

不存在多线程竞争（多线程加锁时间错开），减少重量级锁的性能消耗

**流程** 每个线程都的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的Mark Word，加锁时让锁记录中 Object reference指向锁对象，并尝试用cas替换对象的 Mark Word，将Mark Word的值存入锁记录，如果cas替换成功，对象头中存储了锁记录地址和状态00 ，表示由该线程给对象加锁；若cas失败，一种情况是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入锁膨胀过程；另一种情况是自己执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数，当退出 synchronized 代码块（解锁时）如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示重入计数减一；当退出 synchronized 代码块（解锁时）锁记录的值不为 null，这时使用 cas 将 Mark Word 的值恢复给对象头成功，则解锁成功，失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程