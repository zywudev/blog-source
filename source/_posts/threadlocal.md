---
title: 理解 ThreadLocal
date: 2019-02-19 14:16:59
tags: 
- Android
- Java
---

## ThreadLocal 是什么
ThreadLocal 提供了线程局部变量。它和普通变量的区别在于，普通变量可以被任何一个线程访问并修改，而使用 ThreadLocal 创建的变量只能被当前线程访问，也就是线程私有，其他线程无法访问和修改。

## ThreadLocal 用法

```java
// 定义一个 ThreadLocal 对象
private ThreadLocal<Boolean> threadLocal = new ThreadLocal<>();

// 分别在主线程、子线程1和子线程2中设置和访问它的值
threadLocal.set(true);
Log.e(TAG, "[Thread#main]threadLocal=" + threadLocal.get() );
new Thread("Thread#1"){
    @Override
    public void run() {
        threadLocal.set(false);
        Log.e(TAG, "[Thread#1]threadLocal=" + threadLocal.get() );
    }
}.start();

new Thread("Thread#2"){
    @Override
    public void run() {
        Log.e(TAG, "[Thread#2]threadLocal=" + threadLocal.get() );
    }
}.start();
```

在上面的代码中，在主线程中设置 threadLocal 的值为 true，子线程1中设置 threadLocal 的值为 false，子线程2中未设置 threadLocal 的值。

输出结果如下，可以看到，虽然在不同线程中访问的是同一个 ThreadLocal 对象，但是它们通过 ThreadLocal 获取的值却是不一样的。

```java
[Thread#main]threadLocal=true
[Thread#1]threadLocal=false
[Thread#2]threadLocal=null
```

## ThreadLocal 原理

ThreadLocal 内部是如何实现的，我们从源码中一探究竟。

从 `set` 方法开始，主要工作是

- 获取当前线程
- 获取或当前线程的 ThreadLocalMap 对象
- 如果 ThreadLocalMap 不为空，设置值；否则创建 ThreadLocalMap 对象并设置值

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

`getMap` 方法中获取 ThreadLocalMap 的方法

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

这个方法获取得实际是 Thread 对象的 threadLocals 变量

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

如果是初次调用 `set` 方法，则 ThreadLocalMap 对象为空，会去创建 ThreadLocalMap，并设置初始值。

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

所以可以总结下 ThreadLocal 的设计思路：每个 Thread 维护一个 ThreadLocalMap 映射表，这个映射表的 key 是 TreadLocal 实例本身，value 是真正存储的值。

## ThreadLocalMap

构造 ThreadLocalMap 的主要过程：

- 初始化存放 Entry 对象的数组
- 通过 key（ThreadLocal 类型）的 hashcode 计算存储的索引位置
- 在指定索引位置存放 Entry 对象
- 记录数组中 Entry 对象的个数
- 设定数组扩展阈值

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY]; 
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue); 
    size = 1;
    setThreshold(INITIAL_CAPACITY); 
}
```

下面来看一下 Entry 的结构：

```java
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
}
```

Entry 是 ThreadLocalMap 的静态内部类，继承自 `WeakReference<ThreadLocal>`，从`super(k)` 可以看出 Entry 是一个对 ThreadLocal 的弱引用。另外，Entry 包含了对 value 的强引用。

## ThreadLocal 内存泄漏的问题

首先绘制了 ThreadLocal 相关的对象引用内存图：







