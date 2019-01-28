---
title: JVM 七种垃圾收集器
date: 2019-01-17 20:43:05
tags: JVM
---

Java 垃圾收集器是 [垃圾收集算法](http://wuzhangyang.com/2019/01/15/garbage-collection-algorithm/) 的具体实现。

下图展示的是 7 种作用于不同分代的收集器，如果两种收集器之前有连接，表示它们可以配合使用。收集器所在的位置表示它是属于新生代收集器还是老年代收集器。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/seven_garbage_collector.png)

## 1、 Serial 收集器

**单线程**、**串行**收集器。即在垃圾清理时，必须暂停其他所有工作线程。

它是采用**复制算法**的**新生代收集器**。

下图是 Serial 收集器的运行过程。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/serial_collector.png)

## 2、ParNew 收集器

ParNew 收集器是 Serial 收集器的多线程版本。除了使用多线程收集，其他与 Serial 收集相比并无太多创新之处。

默认开启的线程数量与 CPU 数量相同。

在单 CPU 的环境，ParNew 收集器不会比 Serial 收集器更优秀。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/parnew_collector.png)

## 3、Parallel Scavenge 收集器

Parallel Scavenge 收集器也是一个 并行的多线程新手代收集器，使用的是复制算法。

特点在于它的目标是达到一个可控制的吞吐量（Throughput）。

吞吐量就是 CPU 用于运行用户代码得时间与 CPU 消耗时间的比值。

吞吐量 = 运行用户代码时间 / (运行用户代码时间 + 垃圾收集时间)

高吞吐量可以高效率地利用 CPU 时间，尽快完成程序地运行任务，适合在后台运行不需要太多交互的任务。

-XX:GCTimeRatio : 设置吞吐量大小。

-XX:MaxGCPauseMillis : 设置最大垃圾收集停顿时间。

## 4、Serial Old 收集器

Serial 收集器的老年代产品。同样是单线程，使用标记整理算法。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/serial_collector.png)

## 5、Parallel Old 收集器

Parallel Old 是 Parallel Scanvenge 的老年代版本，使用多线程和标记整理算法。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/parallel_old_collector.png)

## 6、CMS 收集器

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。

从名称是上看出 CMS 采用的是标记清除算法。整个过程有四个步骤：

- 初始标记（CMS initial mark）：仅仅标记一下 GC Roots 能关联到的对象，速度很快。
- 并发标记（CMS concurrent mark）：GC Roots Tracing 过程。
- 重新标记（CMS remark）：修正并发标记期间引用变化那一部分对象
- 并发清除（CMS concurrent sweep）

其中，初始标记、重新标记需要“Stop The World”。并发标记和并发清除时收集器线程可以与用户线程一起工作。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/cms_collector.png)

**优势**：

并发收集、低停顿。

**缺陷**：

- 对 CPU 资源敏感。多线程导致占用一部分 CPU 资源而导致应用程序变慢。
- 无法处理**浮动垃圾**。并发清理过程中用户线程还在运行，会产生新的垃圾，CMS 无法在当次收集中处理它们，只好等待下一次 GC 时再清理掉。这一部分垃圾称为浮动垃圾。
- CMS 采取的标记清除算法会产生大量空间碎片。往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。

## 7、G1收集器

**Region**

上述的 GC 收集器将连续的内存空间划分为新生代、老生代和永久代（JDK 8 去除了永久代，引入了元空间 Metaspace），这种划分的特点是各代的存储地址（逻辑地址）是连续的。

G1 (Garbage First) 的各代存储地址是不连续的，每一代都使用了 n 个不连续的大小相同的 region， 每个 region 占有一块连续的虚拟内存地址。

**G1 将堆分为多个大小相等的独立区域（Region），新生代和老生代不再物理隔离。**

**可预测的停顿时间**

G1 跟踪各个 Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。

**避免全堆扫描**

多个 Region 之前的对象可能会有引用关系，在做可达性分析时需要扫描整个堆才能保证准确性，这显然对降低了 GC 效率。

为避免全堆扫描，虚拟机为 G1 中每个 Region 维护了一个与之对应的 Remembered Set。虚拟机发现程序在对Reference类型的数据进行写操作时，会产生一个Write Barrier暂时中断写操作，检查 Reference 引用的对象是否处于不同的 Region 之中（在分代的例子中就是检查是否老年代中的对象引用了新生代中的对象），如果是，便通过 CardTable **把相关引用信息记录到被引用对象所属的Region的Remembered Set之中**。当进行内存回收时，在GC根节点的枚举范围中加入 Remembered Set 即可保证不对全堆扫描也不会有遗漏。

**G1 的运作步骤**：

- 初识标记（Initial Marking）
- 并发标记（Concurrent Marking）
- 最终标记（Final Marking）
- 筛选回收（Live Data Counting and Evacuation）

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/g1_collector.png)



## 参考资料

[深入理解Java虚拟机:JVM高级特性与最佳实践(第2版)](https://book.douban.com/subject/24722612/)

