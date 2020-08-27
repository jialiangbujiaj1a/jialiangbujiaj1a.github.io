---
title: Synchronized和Lock
tags: Synchronized和Lock ReentrantLock
sidebar: 
  nav: docs-zh
---

### ReentrantLock可重入锁

ReentrantLock 将由最近成功获得锁定，并且还没有释放该锁定的线程所拥有。当锁定没有被另一个线程所拥有时，调用 lock 的线程将成功获取该锁定并返回。如果当前线程已经拥有该锁定，此方法将立即返回。

ReentrantLock还提供了公平锁也非公平锁的选择，构造方法接受一个可选的公平参数（默认非公平锁），当设置为true时，表示公平锁，否则为非公平锁。公平锁与非公平锁的区别在于公平锁的锁获取是有顺序的。但是公平锁的效率往往没有非公平锁的效率高。

### ReentrantReadWriteLock读写锁

 读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。 
 
 写锁就是一个支持可重入的排他锁。读锁为一个可重入的共享锁，它能够被多个线程同时持有，在没有其他写线程访问时，读锁总是或获取成功。
 
 读写锁ReentrantReadWriteLock实现接口ReadWriteLock，该接口维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。
 
 ReadWriteLock定义了两个方法。readLock()返回用于读操作的锁，writeLock()返回用于写操作的锁。
 
 ReentrantReadWriteLock与ReentrantLock一样，其锁主体依然是Sync，它的读锁、写锁都是依靠Sync来实现的。所以ReentrantReadWriteLock实际上只有一个锁，只是在获取读取锁和写入锁的方式上不一样而已，它的读写锁其实就是两个类：ReadLock、writeLock，这两个类都是lock实现。 在ReentrantLock中使用一个int类型的state来表示同步状态，该值表示锁被一个线程重复获取的次数。但是读写锁ReentrantReadWriteLock内部维护着两个一对锁，需要用一个变量维护多种状态。所以读写锁采用“按位切割使用”的方式来维护这个变量，将其切分为两部分，高16为表示读，低16为表示写。分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过为运算。假如当前同步状态为S，那么写状态等于 S & 0x0000FFFF（将高16位全部抹去），读状态等于S >>> 16(无符号补0右移16位)。
 

### 多线程卖票

#### Synchronized 实现

```
public class SaleTicketsWithSynchronized {

    public static void main(String[] args) {
        SaleTicket saleTicket = new SaleTicket();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    saleTicket.saleTicket();
                }
            }
        }, "A").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    saleTicket.saleTicket();
                }
            }
        }, "B").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    saleTicket.saleTicket();
                }
            }
        }, "C").start();

    }

}

class SaleTicket {
    private int total = 100;

    public synchronized void saleTicket() {
        if (total > 0) {
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "卖出第" + (total--) + "还剩下" + total);
        }
    }
}
```

#### Lock 实现

```
public class SaleTicketsWithLock {
    public static void main(String[] args) {
        SaleTicket saleTicket = new SaleTicket();
        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                saleTicket.saleTicket();
            }
        }, "A");

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                saleTicket.saleTicket();
            }
        }, "B");

        new Thread(() -> {
            for (int i = 0; i < 40; i++) {
                saleTicket.saleTicket();
            }
        }, "C");


    }
}

class SaleTicksLock {
    private int number = 30;
    private Lock lock = new ReentrantLock();

    public void sale() {
        lock.lock();
        try {
            if (number > 0) {
                System.out.println(Thread.currentThread().getName() + "卖出第" + (number--) + "还剩下" + number);
            }
        } catch (Exception e) {

        } finally {
            lock.unlock();
        }
    }
}
```

### Synchronized和Lock区别

1、synchronized 关键字，Java内置的。lock 是一个Java类 

2、synchronized 无法判断是否获取锁、lock 可以判断是否获得锁

3、synchronized 锁会自动释放！ lock 需要手动在 finally 释放锁，如果不释放锁，就会死锁

4、synchronized 线程1阻塞 线程2永久等待下去。 lock可以 lock.tryLock(); 尝试获取锁，如果尝试获取不到锁，可以结束等待

5、synchronized 可重入，不可中断，非公平的，Lock锁，可重入、可以判断、可以公平！

### Synchronized方法锁、对象锁、类锁区别

#### 方法锁
每个类的对象对应一个锁，当对象中的某个方法被synchronized修饰后，调用该方法的时候必须获得该对象的“锁”。该方法一旦执行就会占有该锁，别的线程使用该对象调用这个方法的时候就会被阻塞直到这个方法执行完后释放锁，被阻塞的线程才能获得锁，从而进入执行状态。这种机制确保了在同一时刻，对于每一个对象的实例，其所有声明为synchronized方法中最多只有一个处于可执行状态。从而避免了类成员变量的访问冲突。

#### 对象锁
当一个对象中有synchronized方法或者sychronized代码块的时候。调用此对象方法的时就必须获取对象锁，如果此对象的锁已被别的线程获取，那么就必须等待别的线程释放锁后才可以 执行该方法（方法锁也是对象锁）。方法锁和对象锁都差不多，方法锁针对的是一个方法。对象锁则是一个代码块，针对的是一部分代码。

#### 类锁
synchronized修饰的代码块或者synchronized修饰的静态方法。由于一个类不论被实例化多少次，这个类中的静态方法和变量只会被加载和初始化一份，一旦某个静态方法被修饰为synchronized，此类的所有实例化对象共用一把锁，称之为类锁。