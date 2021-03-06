---
title: Java 垃圾收集算法有哪些？
date: 2019-01-15 14:13:32
tags: 
- JVM
- GC
- Java
categories: Java
---

本文主要介绍几种 Java 垃圾收集算法的原理及其优缺点。

## 标记清除（Mark-Sweep）算法

首先进行标记工作，标识出所有要回收的对象，然后进行统一回收被标记的对象。

对象标记的过程在 [Java 对象的自我救赎](http://wuzhangyang.com/2019/01/14/java-object-self-redemption/) 一文中有介绍。执行过程如下图：

![mark_sweep](garbage-collection-algorithm/mark_sweep.png)

**它的不足之处在于**：

1、标记、清除的效率都不高。

2、清除后产生大量的内存碎片，空间碎片太多会导致在分配大对象时无法找到足够大的连续内存，从而不得不触发另一次垃圾回收动作。

## 复制（Copying）算法

将可用内存按容量分成大小相等的两块，每次只使用其中的一块。

当这一块内存用完了，就将还存活的对象复制到另外一块上面，再把已使用过的内存空间一次清理掉。

**商用虚拟机都采用这种算法回收新生代的对象**。因为新生代的对象每次回收都基本上只有 10% 左右的对象存活，需要复制的对象少，效率高。执行过程如下图：

![copying](garbage-collection-algorithm/copying.png)

**优点：**

因为是对整个半区进行内存回收，内存分配时不用考虑内存碎片等情况。实现简单，效率较高。

**不足之处：**

既然要复制，需要提前预留内存空间，有一定的浪费。

在对象存活率较高时，需要复制的对象较多，效率将会变低。

## 标记整理（Mark-Compact）算法

与标记清除算法类似，但不是在标记完成后对可回收对象进行清理，而是将所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。执行过程如下图：

![mark_compact](garbage-collection-algorithm/mark_compact.png)

**优点：**

消除了标记清除导致的内存分散问题，也消除了复制算法中内存减半的高额代价。

**不足之处：**

效率低下，需要标记所有存活对象，还要标记所有存活对象的引用地址。效率上低于复制算法。

## 分代收集（Generational Collection）算法

根据对象存活周期的不同将内存划分为几块。对不同周期的对象采取不同的收集算法。

新生代：每次垃圾收集会有大批对象回收，所以采取复制算法。

老年代：对象存活率高，采取标记清理或者标记整理算法。

## 参考

[深入理解Java虚拟机:JVM高级特性与最佳实践(第2版)](https://book.douban.com/subject/24722612/)

