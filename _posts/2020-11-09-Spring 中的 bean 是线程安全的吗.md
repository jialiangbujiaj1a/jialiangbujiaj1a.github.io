---
title: Spring中的bean线程安全问题
tags: Spring中的bean线程安全问题
sidebar:
  nav: docs-zh
---

结论：不是线程安全的

下面总结一下：
1、在@Controller/@Service等容器中，默认情况下，scope值是单例-singleton的，也是线程不安全的。

2、尽量不要在@Controller/@Service等容器中定义静态变量，不论是单例(singleton)还是多实例(prototype)他都是线程不安全的。

3、默认注入的Bean对象，在不设置scope的时候他也是线程不安全的。

4、一定要定义变量的话，用ThreadLocal来封装，这个是线程安全的