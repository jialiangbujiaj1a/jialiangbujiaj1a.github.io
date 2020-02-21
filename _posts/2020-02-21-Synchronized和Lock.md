---
title: Synchronized和Lock
tags: Synchronized和Lock
sidebar: 
  nav: docs-zh
---

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