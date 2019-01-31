---
title: Java 内存分配和回收策略
date: 2019-01-29 14:47:59
tags: JVM
---

Java 的内存分配主要是在程序运行时给对象在堆上分配内存。通常将堆内存结构按新生代和老年代进行划分，堆内存结构图如下：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_heap_allocation.png)

## 新生代

大部分对象创建和销毁的区域。

内部包含 Eden 区域，作为对象初始分配的区域；两个 Survivor，也叫 from、to 区域，用来放置从 Minor GC 中生存下来的对象。

**TLAB**

对 Eden 区域再进行划分， Hotspot JVM 还有一个概念叫着 Thread Local Allocation（TLAB），这是 JVM 为每个线程分配的一个私有缓存区域。多线程同时分配内存时，为了避免操作同一地址，可能需要使用加锁机制，进而影响分配速度。TLAB 能够解决这个问题。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_tlab.png)

start、end 就是每个 TLAB 的起始结束指针，top 则表示已经分配到哪里。所以在分配新对象时，移动 top，当 top 与 end 相遇，表示该缓存已经满了， JVM 会试图再从 Eden 里分配一块。

## 老年代

**大对象直接进入老年代**

对象先在 TLAB 上分配内存，如果 TLAB 空间不足，会在 Eden 区域给对象分配空间，但是如果对象太大，无法在新生代找到足够长的连续空闲空间， JVM 会直接将对象分配到老年代。

这里的大对象比如是较大的字符串或者数组，因此在写程序时避免分配“朝生夕死”的大对象。

**长期存活的对象直接进入老年代**

在经历了多次 Minor GC 后仍然存活的对象，如果对象的年龄达到老年代阈值，会直接进入老年代。下文会阐述。

**动态对象年龄判定**

为了适应不同程序的内存情况，虚拟机不是永远只在对象的年龄达到老年代阈值时才将对象晋升到老年代。

如果在 Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于等于该年龄的对象就可以直接进入老年代。

## 永久代

早期 JVM 的方法区实现，储存 Java 常量池、类元数据等，在 JDK 8 之后取消了永久代。

## 堆内存参数 

最大堆体积

```jav
-Xmx:value
```

最小堆体积

```java
-Xms:value
```

 老年代和新生代的比例。

```java
-XX：NewRatio=value
```

默认情况，老年代是新生代的 2 倍。即 新生代是堆大小的 1/3。也可以直接调整新生代的大小。

```java
-XX:NewSize=value
```

Eden 和 Survivor 的大小比例。YoungGen = Eden + 2 * Survivor。

```java
-XX：SurvivorRatio=value
```

堆内存结构中每一代中都存在Reserved 区域，当 Xms 小于 Xmx 时，堆的大小不会直接扩展到上限。当内存需求不断增长， JVM 会逐渐扩展区域大小，所以 Reserved 区域表示保留区域，暂时不可用的空间。

## Minor GC

新生代 GC。

Java 应用不断创建对象，优先分配在 Eden 区域，当空间占用达到一定阈值时，触发 Minor GC。没有被引用的对象被回收，仍然存活的对象被复制到 JVM 选择的 Survivor 区域。如下图，数字 1 表示对象的存活年龄计数。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_minor_gc_1.png)

在下一次 Minor GC 时，另外一个 Survivor 区域会成为 to 区域， Eden 区域存活的对象和 from 区域对象都会被复制到 to 区域，存活的年龄计会被加 1。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_minor_gc_2.png)

上述过程会发生很多次，直到有对象年龄计数达到阈值，这些对象会被晋升到老年代。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_minor_gc_3.png)

## Full GC

新生代、老年代和永久代都进行 GC 操作。

**调用System.gc()**

代码中 `System.gc()` 方法的调用是建议 JVM 进行 Full GC，多数情况下会触发 Full GC。

**老年代空间不足**

老年代的对象主要是大对象、长期存活的对象。如果老年代空间不足时，会触发 Full GC。

**空间分配担保失败**

当准备要触发一次 Minor GC 时，如果发现统计数据说之前 Minor GC 的平均晋升大小比目前老年代剩余的空间大，则不会触发 Minor GC 而是转为触发 Full GC。

**JDK 1.7 及以前永久代空间不足**

在JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 class 的信息、常量、静态变量等数据，当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。

在JDK 1.8中用元空间替换了永久代作为方法区的实现，元空间是本地内存，因此减少了一种 Full GC 触发的可能性。

**Concurrent Mode Failure**

执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（有时候“空间不足”是 CMS GC 时当前的浮动垃圾过多导致暂时性的空间不足触发Full GC），便会报 `Concurrent Mode Failure` 错误，并触发 Full GC。

## 参考资料

[深入理解Java虚拟机:JVM高级特性与最佳实践(第2版)](https://book.douban.com/subject/24722612/)

Java 核心技术 36 讲（极客时间）







