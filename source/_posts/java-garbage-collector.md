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

-XX:GCTimeRatio : 设置吞吐量大小

-XX:MaxGCPauseMillis : 设置最大垃圾收集停顿时间。

## 4、Serial Old 收集器

Serial 收集器的老年代产品。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/serial_collector.png)



## 5、Parallel Old 收集器

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/parallel_old_collector.png)

## 6、CMS 收集器

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/cms_collector.png)

## 7、G1收集器

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/g1_collector.png)



## 参考资料

[深入理解Java虚拟机:JVM高级特性与最佳实践(第2版)](https://book.douban.com/subject/24722612/)

