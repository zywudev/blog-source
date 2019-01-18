---
title: JVM 中如何判断对象可以被回收?
date: 2019-01-12 20:34:01
tags:
---

JVM 的垃圾回收器主要关注的是堆上创建的实例对象，在每次对这些对象进行回收前，需要确定哪些对象是可以去进行回收的。

主要有下面两种方法。

## 引用计数算法

给对象添加一个引用计数器，当有一个地方引用它，计数器值加 1；当引用失效时，计数器值减 1。任何时刻计数器值为 0 表示这个对象可以被回收了。

优点：

判断效率高，实现简单。

不足之处：

难以解决对象之间相互循环引用的问题。

比如：

```java
public class GCDemo {
     
    public static void main(String[] args) {
        GCObject objA = new GCObject();  // 1
        GCObject objB = new GCObject();  // 2
        
        objA.instance = objB;  // 3
        objB.instance = objA;  // 4
        
        objA = null;  // 5
        objB = null;  // 6
    }
}

class GCObject {
    public Object instance = null;
}
```

堆栈内存模型如下图：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_judge_object_recycle_1.png)

main 方法中执行的 6 个步骤对应的引用计数结果：

1、实例A 引用计数加 1，引用计数 = 1；

2、实例B 引用计数加 1，引用计数 = 1；

3、实例B 引用计数加 1，引用计数 = 2；

4、实例A引用计数加 1，引用计数 = 2；

5、objA 引用不再指向实例 A，实例 A 的引用计数减为 1；

6、objB 引用计数不再指向实例 B，实例B的引用计数减为 1。

到此，GCObject 的实例 A 和 实例 B 的引用计数都不为 0， 此时如果执行垃圾回收，实例 A 和实例 B 是不会被回收的，也就出现内存泄漏了。

## 可达性分析算法

Java 中采用的是可达性分析算法判断对象是否可以被回收的。

基本思路：

通过一系列称为 "GC Roots" 的对象作为起始点，从这个节点向下搜索，搜索走过的路径就是引用链，当一个对象到 GC Roots 没有任何引用链相连，也就是从 GC Roots 到这个对象不可达，则这个对象不可达，可以被回收。

可作为 GC Roots 的对象有：

- 虚拟机栈中的引用的对象
- 方法区的静态变量和常量引用的对象
- 本地方法栈中 JNI 引用的对象

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/jvm_judge_object_recycle_2.png)

在上面的例子中，当执行第 5、6 步后，虽然实例 A 和实例 B 相互引用，但是它们到 GC Roots 都是不可达的，所以它们都会被判定成可回收对象。

## 参考资料

[深入理解Java虚拟机:JVM高级特性与最佳实践(第2版)](https://book.douban.com/subject/24722612/)

