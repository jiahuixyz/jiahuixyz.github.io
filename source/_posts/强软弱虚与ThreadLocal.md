---
title: 强软弱虚与ThreadLocal
date: 2020-05-16 15:01:44
tags:
---

Java中有四种引用类型：强引用、软引用、弱引用、虚引用。

Java为什么要设计这四种引用
- 可以让程序员通过代码的方式来决定某个对象的生命周期；
- 有利用垃圾回收

软引用
```
SoftReference<Student>studentSoftReference=new SoftReference<Student>(new Student());
```
当内存不足，会触发JVM的GC，如果GC后，内存还是不足，就会把软引用包裹的对象给干掉，也就是只有在内存不足，JVM才会回收该对象。

<!--more-->

弱引用
```
WeakReference<byte[]> weakReference = new WeakReference<byte[]>(new byte[1024*1024*10]);
```
不管内存是否足够，只要发生GC，弱引用包裹的对象都会被回收：

- WeakHashMap。当key只有弱引用时，GC发现后会自动清理键和值，作为简单的缓存表解决方案。

- ThreadLocal。ThreadLocal.ThreadLocalMap.Entry 继承了弱引用，key为当前线程实例，和WeakHashMap基本相同。

虚引用
```
PhantomReference<byte[]> reference = new PhantomReference<byte[]>(new byte[1], queue);
System.out.println(reference.get());//null
```
无法通过虚引用来获取对一个对象的真实引用，虚引用必须与ReferenceQueue一起使用。

当GC准备回收一个对象，如果发现它还有虚引用，就会在回收之前，把这个虚引用加入到与之关联的ReferenceQueue中。

在NIO中，使用虚引用管理堆外内存。

2020我们来谈谈“强软弱虚”四种引用<br>
<https://www.jianshu.com/p/aeb945599fc1>

> 本文部分内容引用网络，侵删。