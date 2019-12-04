---
title: String、StringBuilder 和 StringBuffer
date: 2018-09-09 20:56:56
tags: Java 基础
categories: Java
---

在之前的文章 [Java 中 String 类为什么要设计成不可变的？](http://wuzhangyang.com/2018/09/01/why-string-is-immutable/) 中对 String 的特性已经作了总结。这篇文章主要介绍另外两个常用的类 StringBuilder 和 StringBuffer 的特性。

我们知道 String 是不可变的 (Immutable)，字符串的操作会产生新对象，消耗内存。为此，JDK 提供了 StringBuffer 和 StringBuilder 两个类。

StringBuffer 和 StringBuilder 都实现了 AbstractStringBuilder 抽象类，拥有几乎一致对外提供的接口；它们底层在内存中的存储方式与 String 相同， 都是以一个有序的字符序列进行存储，不同点在于 StringBuffer 和 StringBuilder 对象的值是可以改变的，并且值改变以后，对象的引用不会发生改变。

```java
 public StringBuffer() {
     super(16);
 }

 public StringBuilder() {
     super(16);
 }
```

两者对象在构造时初始字符串长度为16，当超过默认大小后，会创建一个更大的数组，并将原先的数组内容复制过来，再丢弃旧的数组，比较消耗内存。因此，对于较大对象的扩容会涉及大量的内存复制操作，如果能够预先估计指定大小，可以提升性能。

**两者之间的不同点在于： StringBuffer 是线程安全的，StringBuilder 是线程不安全的 。**其中，StringBuffer 的线程安全是通过在 **synchronize** 关键字实现，为此，StringBuffer 的性能远低于 StringBuilder。

在无线程安全问题的情况下，字符串拼接操作有以下两种写法，到底哪一种写法更合理呢？

```java
 StringBuilder strBuilder = new StringBuilder().append("aa")
                .append("bb").append("cc").append("dd");

String myStr = "aa" + "bb" + "cc" + "dd";
```

做个实验，对以下代码进行反编译。

```java
public class Main {
    public static void main(String[] args) {

        String myStr = "aa" + "bb" + "cc" + "dd";

        System.out.println("myStr:" + myStr);

    }
}
```

使用 JDK 8 先编译再反编译：

```java
javac Main.java
javap -v Main.class
```

输出片段为：

```java
0: ldc           #2         // String aabbccdd
2: astore_1
3: getstatic     #3         // Field java/lang/System.out:Ljava/io/PrintStream;
6: new           #4         // class java/lang/StringBuilder
9: dup
10: invokespecial #5        // Method java/lang/StringBuilder."<init>":()V
13: ldc           #6        // String myStr:
15: invokevirtual #7        // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
18: aload_1
19: invokevirtual #7        // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
22: invokevirtual #8        // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
25: invokevirtual #9        // Method java/io/PrintStream.println:(Ljava/lang/String;)V
28: return
```

可以看出来，字符串拼接操作会自动被 javac 转化为 StringBuilder 操作，这是 Java 内部作出的优化。所以在普通清况下，不用过分纠结字符串拼接一定要使用 StringBuilder 实现，毕竟其写法可读性差，需要敲更多代码。

最后简单总结下各自的**应用场景**：

1、在字符串内容不经常发生变化的业务场景优先使用 String 类。

2、在频繁进行字符串的操作，并且需要考虑线程安全的情况下，建议使用 StringBuffer。

3、在频繁进行字符串的操作，无需考虑线程安全的情况下，建议使用 StringBuilder。

 

