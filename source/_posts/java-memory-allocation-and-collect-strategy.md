---
title: Java 内存分配和回收策略
date: 2019-01-29 14:47:59
tags: JVM
---

Java 的内存分配主要是在程序运行时给对象在堆上分配内存。通常将堆内存结构按新生代和老年代进行划分，堆内存结构图如下：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_heap_allocation.png)

**新生代**

大部分对象创建和销毁的区域。

内部包含 Eden 区域，作为对象初始分配的区域；两个 Survivor，也叫 from、to 区域，用来放置从 minor GC 中生存下来的对象。

**TLAB**

对 Eden 区域再进行划分， Hotspot JVM 还有一个概念叫着 Thread Local Allocation（TLAB），这是 JVM 为每个线程分配的一个私有缓存区域。多线程同时分配内存时，为了避免操作同一地址，可能需要使用加锁机制，进而影响分配速度。TLAB 能够解决这个问题。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_tlab.png)

start、end 就是每个 TLAB 的起始结束指针，top 则表示已经分配到哪里。所以在分配新对象时，移动 top，当 top 与 end 相遇，表示该缓存已经满了， JVM 会试图再从 Eden 里分配一块。

**老年代**

放置长生命周期的对象区域。通常是从 Survivor 中拷贝过来的对象。有一种情况，如果对象太大，无法在新生代找到足够长的连续空闲看空间， JVM 会直接将对象分配到老年代。

**永久代**

早期 JVM 的方法区实现，储存 Java 常量池、类元数据等，在 JDK 8 之后取消了永久代。

- Xms：最大堆体积
- Xms：最小堆体积
- -XX：NewRatio 老年代和新生代的比例。默认情况，老年代是新生代的 2 倍。即 新生代是堆大小的 1/3。也可以直接调整新生代的大小（-XX：NewSize）。
- -XX：SurvivorRatio Eden 和 Survivor 的大小比例。YoungGen = Eden+ 2 * Survivor。
- 堆内存结构中每一代中都存在Reserved 区域，当 Xms 小于 Xmx 时，堆的大小不会直接扩展到上限。当内存需求不断增长， JVM 会逐渐扩展区域大小，所以 Reserved 区域表示保留区域，暂时不可用的空间。

**Minor GC**

Java 应用不断创建对象，优先分配在 Eden 区域，当空间占用达到一定阈值时，触发 Minor GC。没有被引用的对象被回收，仍然存活的对象被复制到 JVM 选择的 Survivor 区域。如下图，数字 1 表示对象的存活年龄计数。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_minor_gc_1.png)

在下一次 Minor GC 时，另外一个 Survivor 区域会成为 to 区域， Eden 区域存活的对象和 from 区域对象都会被复制到 to 区域，存活的年龄计会被加 1。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_minor_gc_2.png)

上述过程会发生很多次，直到有对象年龄计数达到阈值，这些对象会被晋升到老年代。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_minor_gc_3.png)

老年代的 GC 叫着 Major GC，对整个堆进行的清理叫着 Full GC。























