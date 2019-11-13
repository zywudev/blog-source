---
title: Andoid面试题（5）：谈谈 Handler 机制和原理？
date: 2019-11-13 18:52:07
tags: Android面试题
---

> 这一系列文章致力于为 Android 开发者查漏补缺，准备面试。
>
> 所有文章首发于公众号「JaqenAndroid」，长期持续更新。
>
> 由于笔者水平有限，总结的内容难免会出现错误，欢迎留言指出，大家一起学习、交流、进步。

## 1、说一下 Handler 消息机制中涉及到哪些类，各自的功能是什么？

Handler 主要用于跨线程通信。涉及MessageQueue/Message/Looper/Handler 这 4 个类。 

- Message：消息，分为硬件产生的消息和软件生成的消息。

- MessageQueue：消息队列，主要功能是向消息池投递信息 (`MessageQueue.enqueueMessage`) 和取走消息池的信息 (`MessageQueue.next`) 。

- Handler：消息处理者，负责向消息池中发送消息 (`Handler.enqueueMessage`) 和处理消息 (`Handler.handleMessage`) 。

- Looper：消息泵，不断循环执行 (`Looper.loop`) ，按分发机制将消息分发给目标处理者。

它们之间的类关系：

Looper 有一个 MessageQueue 消息队列；MessageQueue 有一组待处理的 Message；Message 中有一个用于处理消息的 Handler；Handler 中有 Looper 和 MessageQueue。

![图片来源 gityuan](android-interview-5-handler/handler_main.jpg)

## 2、一个线程可以有几个 Looper、几个 MessageQueue 和几个 Handler？

在 Android 中，Looper 类利用了 ThreadLocal 的特性，保证了每个线程只存在一个 Looper 对象。

关于 ThreadLocal 可以看这篇文章：[理解 ThreadLocal]( http://wuzhangyang.com/2019/02/19/threadlocal/ )

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

Looper 构造函数中创建了 MessageQueue 对象，因此一个线程只有一个 MessageQueue。

```java
private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
}
```

可以有多个 Handler。

Handler 在创建时与 Looper 和 MessageQueue 关联起来：

```java
public Handler(Callback callback, boolean async) {
    ...
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    ...
}
```

Handler 发送消息是将消息传递给 MessageQueue：

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

注意 `msg.target = this;`， 这里将当前的 Handler 赋值给 Message 对象，在后面处理消息时就能依据 msg.target 区分不同的 Handler。

## 3、可以在子线程直接创建一个 Handler 吗？会出现什么问题，那该怎么做？

不能在子线程直接 new 一个 Handler。因为 Handler 的工作依赖于 Looper，而 Looper 又是属于某一个线程的，其他线程不能访问，所以在线程中使用 Handler 时必须要保证当前线程中 Looper 对象并且启动循环。不然会抛出异常。

```java
throw new RuntimeException("Can't create handler inside thread " + Thread.currentThread() + " that has not called Looper.prepare()");
```

正确做法是：

```java
class LooperThread extends Thread {
    public Handler mHandler;

    public void run() {
        Looper.prepare();   // 为线程创建 Looper 对象

        mHandler = new Handler() {  
            public void handleMessage(Message msg) {
               
            }
        };

        Looper.loop();   // 启动消息循环
    }
}
```

## 4、既然线程中创建 Handler 时需要 Looper 对象，为什么主线程不用调用 `Looper.prepare()` 创建 Looper 对象？

在 App 启动的时候系统默认启动了一个主线程的 Looper（ActivityThread 的 `main` 方法中），`Loop.prepareMainLooper` 方法也是调用了 `Looper.prepare`方法，里面会创建一个不可退出的 Looper, 并 `set` 到 sThreadLocal 对象当中。 

```java
public static void main(String[] args) {
    Looper.prepareMainLooper();
    Looper.loop();
}
```



## 5、 Looper 死循环为什么不会导致应用卡死，会消耗大量资源吗？ 

引用 Gityuan :

> 对于线程即是一段可执行的代码，当可执行代码执行完成后，线程生命周期便该终止了，线程退出。而对于主线程，我们是绝不希望会被运行一段时间，自己就退出，那么如何保证能一直存活呢？简单做法就是可执行代码是能一直执行下去的，死循环便能保证不会被退出，例如，binder 线程也是采用死循环的方法，通过循环方式不同与 Binder 驱动进行读写操作，当然并非简单地死循环，无消息时会休眠。但这里可能又引发了另一个问题，既然是死循环又如何去处理其他事务呢？通过创建新线程的方式。真正会卡死主线程的操作是在回调方法 onCreate/onStart/onResume 等操作时间过长，会导致掉帧，甚至发生ANR，looper.loop本身不会导致应用卡死。
>
> 

> 主线程的死循环一直运行是不是特别消耗CPU资源呢？ 其实不然，这里就涉及到 Linux pipe/epoll 机制，简单说就是在主线程的 MessageQueue 没有消息时，便阻塞在 Loop 的 queue.next() 中的 nativePollOnce() 方法里，此时主线程会释放 CPU 资源进入休眠状态，直到下个消息到达或者有事务发生，通过往 pipe 管道写端写入数据来唤醒主线程工作。这里采用的epoll机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步 I/O，即读写是阻塞的。 所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量 CPU 资源。

详细解答：[Android中为什么主线程不会因为Looper.loop()里的死循环卡死？]( https://www.zhihu.com/question/34652589 )

## 6、 MessageQueue 是队列吗？它是什么数据结构？ 

MessageQueue 不是队列，它内部使用一个 Message 链表实现消息的存和取。 链表的排列依据是  `Message.when`，表示 Message 期望被分发的时间，该值是 `SystemClock. uptimeMillis()` 与 `delayMillis` 之和。 

##7、 `handler.postDelayed()` 函数延时执行计时是否准确？

当上一个消息存在耗时任务的时候，会占用延时任务执行的时机，实际延迟时间可能会超过预设延时时间，这时候就不准确了。

##8、 你对 IdleHandler 有多少了解? 

IdleHandler 是一个接口， 这个接口方法是在消息队列全部处理完成后或者是在阻塞的过程中等待更多的消息的时候调用的，返回值 false 表示只回调一次，true 表示可以接收多次回调。 

```java
Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
    @Override
    public boolean queueIdle() {
        return false;
    }
});
```

##9、 你了解 HandlerThread 吗?

HandlerThread 继承自 Thread，它是一种可以使用 Handler 的 Thread，它的实现也很简单，在 `run`方法中也是通过 `Looper.prepare()` 来创建消息队列，并通过`Looper.loop()`来开启消息循环（与我们手动创建方法基本一致），这样在实际的使用中就允许在 HandlerThread 中创建 Handler 了。

```java
public class HandlerThread extends Thread {
    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
}
```

由于 HandlerThread 的`run`方法是一个无限循环，因此当不需要使用的时候通过`quit`或者`quitSafely`方法来终止线程的执行。

##10、 你对 `Message.obtain()` 了解吗, 或者你知道怎么维护消息池吗 ？

 `Message.obtain()`  是从消息池取 Message，消息池其实是使用 Message 链表结构实现，消息池默认最大值 50。 `Message.obtain()`  每次都是把消息池表头的 Message 取走 ，再把表头指向 next。

```java
public static Message obtain() {
    synchronized (sPoolSync) {
        if (sPool != null) {
            Message m = sPool;
            sPool = m.next;
            m.next = null;  //从sPool中取出一个Message对象，并消息链表断开
            m.flags = 0; // 清除in-use flag
            sPoolSize--; //消息池的可用大小进行减1操作
            return m;
        }
    }
    return new Message(); // 当消息池为空时，直接创建Message对象
}
```

消息在 loop 中被 handler 分发消费之后会执行回收的操作，将该消息内部数据清空并添加到消息链表的表头。 

```java
public void recycle() {
    if (isInUse()) { //判断消息是否正在使用
        if (gCheckRecycle) { //Android 5.0以后的版本默认为true,之前的版本默认为false.
            throw new IllegalStateException("This message cannot be recycled because it is still in use.");
        }
        return;
    }
    recycleUnchecked();
}

//对于不再使用的消息，加入到消息池
void recycleUnchecked() {
    //将消息标示位置为IN_USE，并清空消息所有的参数。
    flags = FLAG_IN_USE;
    what = 0;
    arg1 = 0;
    arg2 = 0;
    obj = null;
    replyTo = null;
    sendingUid = -1;
    when = 0;
    target = null;
    callback = null;
    data = null;
    synchronized (sPoolSync) {
        if (sPoolSize < MAX_POOL_SIZE) { //当消息池没有满时，将Message对象加入消息池
            next = sPool;
            sPool = this;
            sPoolSize++; //消息池的可用大小进行加1操作
        }
    }
}
```

最后，关于 Handler 的详细分析推荐阅读 Gityuan 的文章。

[Android消息机制1-Handler(Java层)]( http://gityuan.com/2015/12/26/handler-message-framework/ )

[Android消息机制2-Handler(Native层)]( http://gityuan.com/2015/12/27/handler-message-native/) 

[Android消息机制3-Handler(实战)]( http://gityuan.com/2016/01/01/handler-message-usage/ )