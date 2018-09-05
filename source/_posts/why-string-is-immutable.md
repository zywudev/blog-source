---
title: Java 中 String 类为什么要设计成不可变的？
date: 2018-09-01 20:18:41
tags:
---

String 是 Java 中不可变的类，所以一旦被实例化就无法修改。不可变类的实例一旦创建，其成员变量的值就不能被修改。本文总结下 String 类设计成不可变的原因及好处，以及 String 类是如何设计成不可变的。

## String 类设计成不可变的原因及好处？

其实好处就是原因，String 设计成不可变，主要是从性能和安全两方面考虑。

#### 1、常量池的需要

这个方面很好理解，Java 中的字符串常量池的存在就是为了性能优化。

字符串常量池（String pool）是 Java 堆内存中一个特殊的存储区域，当创建一个 String 对象时，假如此字符串已经存在于常量池中，则不会创建新的对象，而是直接引用已经存在的对象。这样做能够减少 JVM 的内存开销，提高效率。

比如引用 s1和 s2 都是指向常量池的同一个对象 "abc"，如果 String 是可变类，引用 s1 对 String 对象的修改，会直接导致引用 s2 获取错误的值。

```java
String s1 = "abc";
String s2 = "abc";
```

![](http://om9o63aks.bkt.clouddn.com/FhDNgVxqIlP73m67Cej0-vPDIPkP)

所以，如果字符串是可变的，那么常量池就没有存在的意义了。

#### 2、hashcode 缓存的需要

因为字符串不可变，所以在它创建的时候 hashcode 就被缓存了，不需要重新计算。这就使得字符串很适合作为 HashMap 中的 key，效率大大提高。

#### 3、多线程安全

多线程中，可变对象的值很可能被其他线程改变，造成不可预期的结果。而不可变的 String 可以自由在多个线程之间共享，不需要同步处理。

## String 类是如何实现不可变的？

#### 1、私有成员变量

String 的内部很简单，有两个私有成员变量

```java
/** The value is used for character storage. */
private final char value[];

/** Cache the hash code for the string */
private int hash; // Default to 0
```

而并没有对外提供可以修改这两个属性的方法。

#### 2、Public 的方法都是复制一份数据

String 有很多 public 方法，每个方法都将创建新的 String 对象，比如 substring 方法：

```java
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

#### 3、String 是 final 的

String 被 final 修饰，因此我们不可以继承 String，因此就不能通过继承来重写一些方法。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
}
```

#### 4、构造函数深拷贝

当传入可变数组 value[] 时，进行 copy 而不是直接将 value[] 复制给内部变量。

```java
public String(char value[]) {
    this.value = Arrays.copyOf(value, value.length);
}
```

从 String 类的设计方式，我们可以总结出实现不可变类的方法：

- 将 class 自身声明为 final，这样别人就不能通过扩展来绕过限制了。
- 将所有成员变量定义为 private 和 final，并且不要实现 setter 方法。
- 通过构造对象时，成员变量使用深拷贝来初始化，而不是直接赋值，这是一种防御措施，因为你无法确定输入对象不被其他人修改。
- 如果确实需要 getter 方法，或者其他可能返回内部状态的方法，使用 copy-on-write 原则，创建私有的 copy。







