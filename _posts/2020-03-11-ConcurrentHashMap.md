---
title: ConcurrentHashMap
tags: ConcurrentHashMap
sidebar:
  nav: docs-zh
---

### 原理

在java8之前ConcurrentHashMap是使用分段锁来实现并发的，数据结构为hashmap（数组加链表）的基础上再套一层segment数组，锁加在segment元素上。java8实现了粒度更细的加锁，去掉了segment数组，直接使用synchronized锁住hash后得到的数组下标位置中的第一个元素 ，如下图，这样加锁比segment加锁能支持更高的并发量。

数组里每个位置元素进行put时，都是有一个不同的锁，假如此时有两个线程在同一位置进行put，采取CAS策略，同一时间只有一个线程能成功执行CAS操作，如果很多线程对数组的不同位置进行put，则可以并发执行。

![ConcurrentHashMap](https://jialiangbujiaj1a.github.io/imgs/map/ConcurrentHashMap.jpg)

[ConcurrentHashmap](https://blog.csdn.net/ddxd0406/article/details/81389583)


