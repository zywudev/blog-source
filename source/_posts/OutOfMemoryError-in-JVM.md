---
title: JVM 中的内存溢出
date: 2019-01-11 11:18:23
tags:
---

内存溢出，通俗一点，就是 JVM 内存不足了，没有空闲内存，并且垃圾收集器也无法提供更多内存。

这里的意思是说，通常在抛出 OutOfMemoryError 之前，垃圾收集器会被触发，尽其所能去清理空间。

但也不是在所有情况下垃圾回收器都会被触发，比如分配了一个大对象，超过了堆的最大值，JVM 可能判断出垃圾收集并不能解决这个问题，直接抛出 OutOfMemoryError 。

在 [JVM内存结构](http://wuzhangyang.com/2019/01/10/JVM-memory-structure/) 中，除了程序计数器，其他区域都有可能发生 OutOfMemoryError 。

## 堆溢出

通过`-Xms` 和`Xmx`分别设定堆最小值和最大值。

错误信息：

```java
java.lang.OutOfMemoryError: Java heap space
```

可能原因：

- 内存泄漏
- 堆的大小不合理，比如处理可观的数据量，但是没有显示指定 JVM 堆大小或者指定数值太小
- JVM 处理引用不及时，导致堆积起来，内存无法释放

## 栈溢出

通过 `--Xss` 设置栈容量大小。

这里的栈包括虚拟机栈和本地方法栈。

比如递归操作，没有退出条件，会导致不断的压栈，JVM 就会抛出 StackOverFlowError。

如果 JVM 试图去扩展栈空间的时候失败，则会抛出 OutOfMemoryError。

## 方法区溢出

通过 `-XX:PermSize` 和 `-XX:MaxPermSize` 限制方法区的大小。

`String.intern()` 的作用是：如果字符串常量池中已经包含一个等于此 String对象的字符串，则返回代表池中这个字符串的 String 对象，否则，将此 String 对象包含的字符串添加到常量池中，并且返回此 String 对象的引用。所以，当字符串缓存占用太多空间，也会导致 OOM 问题。

错误信息：

```java
java.lang.OutOfMemoryError: PermGen space
```

JDK 1.7 后，方法区引入元数据区，元数据区默认自增，方法区内存不再那么窘迫。

元数据区错误信息：

```java
java.lang.OutOfMemoryError: Metaspace
```

## 直接内存溢出

通过 `-XX：MaxDirectMemorySize` 指定直接直接内存容量大小。

特征：

Heap Dump 文件中不会看见明显的异常，如果 Dump 文件很小，程序中有使用 NIO，可以考虑检查是否是直接内存溢出。

## 参考资料

[深入理解Java虚拟机:JVM高级特性与最佳实践(第2版)](https://book.douban.com/subject/24722612/)

