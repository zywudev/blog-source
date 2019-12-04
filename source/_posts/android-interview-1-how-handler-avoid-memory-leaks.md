---
title: Android 面试题（1）：使用 Handler 时如何有效地避免内存泄漏问题？
date: 2019-10-23 14:25:59
tags:
- Android面试题
categories: Android面试题
---

> 这一系列文章致力于为 Android 开发者查漏补缺，面试准备。
>
> 所有文章首发于公众号「JaqenAndroid」，长期持续更新。
>
> 由于笔者水平有限，总结的答案难免会出现错误，欢迎留言指出，大家一起学习、交流、进步。

## 什么是内存泄漏？

Java 中采用可达性分析算法判断一个对象是否可被回收。

基本思路是这样的：

通过一系列称为 “GC Roots” 的对象作为起始点，从这个节点向下搜索，搜索走过的路径就是引用链，当一个对象到 GC Roots 没有任何引用链相连，也就是从 GC Roots 到这个对象不可达，则这个对象不可达，可以被回收。

可作为 GC Roots 的对象有：

- 虚拟机栈中的引用的对象

- 方法区的静态变量和常量引用的对象

- 本地方法栈中 JNI 引用的对象

**当一个对象不需要在再使用了，本该被回收时， 而另外一个正在使用的对象持有它的引用从而导致它不能被回收，这就导致本该被回收的对象不能被回收而停留在堆内存中，内存泄漏就产生了。** 

## Handler 是如何造成内存泄漏的？

```java
public class MainActivity extends AppCompatActivity {
    
    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(@NonNull Message msg) {
            // 处理数据
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        loadData();
    }

    private void loadData() {
        // 耗时任务
        // ...
        // 发送数据
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
}
```

 上面是一段简单的 Handler 的使用。 这种方式有可能造成内存泄漏吗？答案是有可能的。我们来分析下造成内存泄漏的原因？

我们知道 Java中非静态内部类会隐式持有外部类的引用，所以这里创建的 Handler 隐式持有外部类 MainActivity 的引用。

而 Handler 一般用来处理后台线程任务的执行结果，如果在线程任务之慈宁宫过程中，用户关闭了 Activity，此时线程尚未执行完，而该线程持有 Handler 的引用，Handler 又持有 Activity 的引用，就导致了 Activity 无法被回收（即内存泄漏）。

还有一种情况，如果你调用 Handler 的 `postDelay()` 方法执行了延时任务， 该方法会将你的Handler 装入一个 Message，并把这条 Message 推到 MessageQueue 中，那么在你设定的 delay 到达之前，会有一条 MessageQueue -> Message -> Handler -> Activity 的链，导致你的 Activity 被持有引用而无法被回收。 

## 如果解决 Handler 导致的内存泄漏问题？

**方法 1、静态内部类 + 弱引用**

既然非静态内部类持有外部类的引用，那么可以将 Handler 声明为静态内部类，Handler 也就不再持有 Activity 的引用，所以 Activity 可以随便被回收。代码如下：

```java
static class MyHandler extends Handler {
    @Override
    public void handleMessage(Message msg) {
        // ...
    }
}
```

此时 Handler 不再持有 Activity 的引用，导致 Handler 无法操作 Activity 中对象，所以可以在 Handler 中添加一个对 Activity 的弱引用（ WeakReference ）：

```java
static class MyHandler extends Handler {
    WeakReference<Activity > mActivityReference;

    MyHandler(Activity activity) {
        mActivityReference= new WeakReference<Activity>(activity);
    }

    @Override
    public void handleMessage(Message msg) {
        final Activity activity = mActivityReference.get();
        if (activity != null) {
            //...
        }
    }
}
```

弱引用的特点是： 在垃圾回收器一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。 所以用户在关闭 Activity 之后，就算后台线程还没结束，但由于仅有一条来自 Handler 的弱引用指向 Activity，Activity 也会被回收掉。这样，内存泄露的问题就不会出现了。 

**方法2： 通过程序逻辑来进行保护**

 在 Activity 被销毁时及时清除消息，从而及时回收 Activity，避免内存泄漏问题。 

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    if (mHandler != null)  {
        mHandler.removeCallbacksAndMessages(null);
    }
}
```







