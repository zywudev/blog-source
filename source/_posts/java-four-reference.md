---
title: Java的强引用，软引用，弱引用，虚引用及其使用场景
date: 2018-09-03 20:13:47
tags:
---

从 JDK1.2 版本开始，把对象的引用分为四种级别，从而使程序能更加灵活的控制对象的生命周期。这四种级别由高到低依次为：强引用、软引用、弱引用和虚引用。

#### 1、强引用（Strong Reference）

强引用就是我们经常使用的引用，其写法如下：

```java
Object o = new Object();
```

**特点**：

- 只要还有强引用指向一个对象，垃圾收集器就不会回收这个对象。
- 显式地设置 o 为 null，或者超出对象的生命周期，此时就可以回收这个对象。具体回收时机还是要看垃圾收集策略。
- 在不用对象的时将引用赋值为 null，能够帮助垃圾回收器回收对象。比如  ArrayList 的 clear() 方法实现：

```java
 public void clear() {
     modCount++;

     // clear to let GC do its work
     for (int i = 0; i < size; i++)
         elementData[i] = null;
     size = 0;
 }
```

#### 2、软引用（Soft Reference）

如果一个对象只具有软引用，在内存足够时，垃圾回收器不会回收它；如果内存不足，就会回收这个对象的内存。

使用场景：

- 图片缓存。图片缓存框架中，“内存缓存”中的图片是以这种引用保存，使得  JVM 在发生 OOM 之前，可以回收这部分缓存。
- 网页缓存。

```java
Browser prev = new Browser();               // 获取页面进行浏览
SoftReference sr = new SoftReference(prev); // 浏览完毕后置为软引用		
if(sr.get()!=null) { 
	rev = (Browser) sr.get();           // 还没有被回收器回收，直接获取
} else {
	prev = new Browser();               // 由于内存吃紧，所以对软引用的对象回收了
	sr = new SoftReference(prev);       // 重新构建
}
```

#### 3、弱引用（Weak Reference）

简单来说，就是将对象留在内存的能力不是那么强的引用。当垃圾回收器扫描到只具有弱引用的对象，不管当前内存空间是否足够，都会回收内存。

**使用场景**：

在下面的代码中，如果类 B 不是虚引用类 A 的话，执行 main 方法会出现内存泄漏的问题， 因为类 B 依然依赖于 A。

```java
public class Main {
    public static void main(String[] args) {

        A a = new A();
        B b = new B(a);
        a = null;
        System.gc();
        System.out.println(b.getA());  // null

    }

}

class A {}

class B {

    WeakReference<A> weakReference;

    public B(A a) {
        weakReference = new WeakReference<>(a);
    }

    public A getA() {
        return weakReference.get();
    }
}
```

在静态内部类中，经常会使用虚引用。例如：一个类发送网络请求，承担 callback 的静态内部类，则常以虚引用的方式来保存外部类的引用，当外部类需要被 JVM 回收时，不会因为网络请求没有及时回应，引起内存泄漏。

#### 4、虚引用（Phantom Reference）

虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

```java
Object obj = new Object();
ReferenceQueue refQueue = new ReferenceQueue();
PhantomReference<Object> phantomReference = new PhantomReference<Object>(obj,refQueue);
```

**使用场景**：

可以用来跟踪对象呗垃圾回收的活动。一般可以通过虚引用达到回收一些非java内的一些资源比如堆外内存的行为。例如：在 DirectByteBuffer 中，会创建一个 PhantomReference 的子类Cleaner的虚引用实例用来引用该 DirectByteBuffer 实例，Cleaner 创建时会添加一个 Runnable 实例，当被引用的 DirectByteBuffer 对象不可达被垃圾回收时，将会执行 Cleaner 实例内部的 Runnable 实例的 run 方法，用来回收堆外资源。