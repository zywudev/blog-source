---
title: Android 面试题（2）：一般什么情况下会导致内存泄漏问题？
date: 2019-10-24 17:30:47
tags: Android面试题
---

> 这一系列文章致力于为 Android 开发者查漏补缺，面试准备。
>
> 所有文章首发于公众号「JaqenAndroid」，长期持续更新。
>
> 由于笔者水平有限，总结的答案难免会出现错误，欢迎留言指出，大家一起学习、交流、进步。

内存泄漏也是面试常见问题，主要可以考察面试者是否了解内存泄漏，工作中是如何排查解决内存泄漏问题，还可以延伸考察 Java 内存回收机制，Java 中对象的引用方式等等。

这篇文章先来介绍下 Android 开发中常见的内存泄漏案例以及相应的解决方案。

## 单例造成的内存泄漏

单例模式在 Android 开发中使用率非常高，但使用不恰当的话也会造成内存泄漏。比如下面的代码：

```java
public class Singleton {

    private static Singleton sInstance;
    private Context mContext;

    private Singleton(Context context) {
        this.mContext = context;
    }

    public static Singleton getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new Singleton(context);
        }
        return sInstance;
    }
}
```

单例类对象的生命周期与应用的周期一样长，如果传入的是 Activity 的 Context，在 Activity 退出时，因单例对象持有 Activity 的引用，导致 Activity 的内存不能被回收，即内存泄漏。

**解决方案**：

1）使用 Application 的 Context，生命周期一致；

2）将短生命周期的属性的引用方式改为弱引用。

## 非静态内部类造成的内存泄漏

非静态内部类持有外部类的引用，如果外部类的实例已经结束生命周期，但内部类仍然在执行，就会导致外部类不能被回收。比如上一期讲解的自定义 Handler 的使用造成的内存泄漏，主要原因 Activity 退出时，Handler 仍然持有 Activity 的引用，导致 Activity 不能被回收。

**解决方案**：

1） 创建一个静态内部类，然后外部类的对象引用使用弱引用；

2）及时关闭耗时或者延时任务，在 Activity 被销毁时及时清除消息，从而及时回收 Activity，避免内存泄漏问题。 

## 系统服务注册未取消造成的内存泄漏

系统服务可以通过 `Context.getSystemService` 获取，它们负责执行某些后台任务，或者为硬件访问提供接口。如果 Context 对象想要在服务内部的事件发生时被通知，那就需要把自己注册到服务的监听器中。然而，这会让服务持有 Activity 的引用，如果在 Activity 的 `onDestory()` 函数中没有释放掉引用就会内存泄漏。 

**解决方案**：

1）使用 Application 的 Context 代替 Activity 的 Context；

2）在 Activity 的 `onDestory()` 方法，调用反注册释放。

## 全局集合类造成的内存泄漏

一般情况下集合类不会造成内存泄漏，但如果是全局性的集合，如果在使用完毕后未进行 remove 清理操作，就很有可能造成内存泄漏，所以在集合不需要的时候要及时清理集合元素。

## 资源未关闭的内存泄漏

 对于使用了 BroadcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap 等资源，应该在 Activity 销毁时及时关闭或者注销。

## WebView 造成的内存泄漏

 WebView 存在内存泄漏的问题，在应用中只要使用一次 WebView，内存就不会被释放掉。

**解决方案**：

为 WebView 开启一个独立的进程，使用 AIDL 与应用的主进程进行通信，WebView 所在的进程可以根据业务的需要选择合适的时机进行销毁，达到正常释放内存的目的。 

## 总结

总的来说就是生命周期长的对象持有了生命周期短的对象，导致生命周期短的对象在回收时无法被释放，就会导致内存泄漏。

了解常见内存泄漏及解决方案，能够帮助我们在开发中尽量少的出现内存泄漏问题。但有些内存泄漏的定位排查比较困难，需要借助一些工具，比如 LeakCanary、MAT 等。内存泄漏的排查定位方法会在后续文章中介绍，欢迎持续关注。