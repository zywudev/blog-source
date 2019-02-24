---
title: Android Handler 消息处理机制
date: 2018-01-07 15:55:35
tags: Android
toc: true
---

日常开发中，一般不会在子线程中直接进行 UI 操作，大部分采取的办法是创建 Message 对象，然后借助 Handler 发送出去，再在 Handler 的 handlerMessage() 方法中获取 Message 对象，进行一系列的 UI 操作。Handler 负责发送 Message， 又负责处理 Message， 其中经历了什么 ，需要从源码中一探究竟。

首先看 Handler 的构造函数：

```java
public Handler(Callback callback, boolean async) {
  if (FIND_POTENTIAL_LEAKS) {
    final Class<? extends Handler> klass = getClass();
    if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
        (klass.getModifiers() & Modifier.STATIC) == 0) {
      Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
            klass.getCanonicalName());
    }
  }

  mLooper = Looper.myLooper();
  if (mLooper == null) {
    throw new RuntimeException(
      "Can't create handler inside thread that has not called Looper.prepare()");
  }
  mQueue = mLooper.mQueue;
  mCallback = callback;
  mAsynchronous = async;
}
```

Handler 的无参构造函数会调用上面的重载构造函数，我们直接看第二个 if 语句， 如果 mLooper 为 null，将会抛出异常 “Can't create handler inside thread that has not called Looper.prepare()”，意思是说如果没有调用 Looper.prepare()， 在当前线程中不能创建 Handler 实例。但是，我们在 UI 主线程中创建 Handler 时，好像并不要调用方法 Looper.prepare()，肯定是系统已经帮我们自动调用了 Looper.prepare() 方法。

我们来看 Looper.myLooper() 方法，这个方法直接从 sThreadLocal 取出 Looper 对象。

```java
public static @Nullable Looper myLooper() {
  return sThreadLocal.get();
}
```

所以 sThreadLocal 中的 Looper 对象肯定是在 Looper.prepare() 方法中 set 进去，Looper.prepare() 方法会调用 Looper.prepare(boolean quitAllowed) 方法。

```java
 private static void prepare(boolean quitAllowed) {
   if (sThreadLocal.get() != null) {
     throw new RuntimeException("Only one Looper may be created per thread");
   }
   sThreadLocal.set(new Looper(quitAllowed));
 }
```

可以看到，这个方法中首先判断当前线程中有没有 Looper 对象了，如果没有，就会创建一个 Looper 对象 set 到 sThreadLocal 中 ；如果有 Looper 对象了，就不能再创建 Looper 对象，即一个线程只能创建一个 Looper 对象。而在 UI 主线程中，系统是什么时候调用了 Looper.prepare() 方法呢？

查看 ActivityThread 中的 main() 方法，代码如下所示：

```java
 public static void main(String[] args) {
   Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
   SamplingProfilerIntegration.start();

   // CloseGuard defaults to true and can be quite spammy.  We
   // disable it here, but selectively enable it later (via
   // StrictMode) on debug builds, but using DropBox, not logs.
   CloseGuard.setEnabled(false);

   Environment.initForCurrentUser();

   // Set the reporter for event logging in libcore
   EventLogger.setReporter(new EventLoggingReporter());

   // Make sure TrustedCertificateStore looks in the right place for CA certificates
   final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
   TrustedCertificateStore.setDefaultUserDirectory(configDir);

   Process.setArgV0("<pre-initialized>");

   Looper.prepareMainLooper(); // 创建 Looper 对象

   ActivityThread thread = new ActivityThread();
   thread.attach(false);

   if (sMainThreadHandler == null) {
     sMainThreadHandler = thread.getHandler();
   }

   if (false) {
     Looper.myLooper().setMessageLogging(new LogPrinter(Log.DEBUG, "ActivityThread"));
   }

   // End of event ActivityThreadMain.
   Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
   Looper.loop();

   throw new RuntimeException("Main thread loop unexpectedly exited");
 }
```

查看 ` Looper.prepareMainLooper();` 这一行，应用启动的时候就调用了这个方法，在这个方法中又去调用了方法 prepare(boolean quitAllowed) 去创建一个 Looper 对象。

```java
public static void prepareMainLooper() {
  prepare(false);
  synchronized (Looper.class) {
    if (sMainLooper != null) {
      throw new IllegalStateException("The main Looper has already been prepared.");
    }
    sMainLooper = myLooper();
  }
}
```

至此，Handler 的创建过程已经清晰。再来看看 Looper 的构造函数如下：

```java
private Looper(boolean quitAllowed) {
  mQueue = new MessageQueue(quitAllowed);
  mThread = Thread.currentThread();
}
```

可以看出在当前线程中创建了一个 MessageQueue，顾名思义，MessageQueue 是用来存放消息的消息队列。同样从这可以看出，一个线程只能创建一个 Looper，也就只有一个 MessageQueue。

那么，Handler 是如何发送消息并处理消息的呢？得从发送消息的方法看起，Handler 提供很多发送消息的方法，但大部分方法最终都会调用 sendMessageAtTime 方法。

```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
  MessageQueue queue = mQueue;
  if (queue == null) {
    RuntimeException e = new RuntimeException(
      this + " sendMessageAtTime() called with no mQueue");
    Log.w("Looper", e.getMessage(), e);
    return false;
  }
  return enqueueMessage(queue, msg, uptimeMillis);
}
```

这个方法的第一个参数是发送的消息，第二个参数是自系统开机到当前时间的毫秒数再加上延迟时间，如果调用的不是 sendMessageDelayed() 方法，延迟时间就为 0。最后调用 enqueueMessage 方法。

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
  msg.target = this;
  if (mAsynchronous) {
    msg.setAsynchronous(true);
  }
  return queue.enqueueMessage(msg, uptimeMillis);
}
```

这个方法中，首先将 msg.target 设置为 handler 对象本身，然后调用 MessageQueue 的 enqueueMessage 方法，我们再看 MessageQueue 的 enqueueMessage 方法。

```java
boolean enqueueMessage(Message msg, long when) {
  if (msg.target == null) {
    throw new IllegalArgumentException("Message must have a target.");
  }
  if (msg.isInUse()) {
    throw new IllegalStateException(msg + " This message is already in use.");
  }

  synchronized (this) {
    if (mQuitting) {
      IllegalStateException e = new IllegalStateException(
        msg.target + " sending message to a Handler on a dead thread");
      Log.w(TAG, e.getMessage(), e);
      msg.recycle();
      return false;
    }

    msg.markInUse();
    msg.when = when;
    Message p = mMessages;
    boolean needWake;
    if (p == null || when == 0 || when < p.when) {
      // New head, wake up the event queue if blocked.
      msg.next = p;
      mMessages = msg;
      needWake = mBlocked;
    } else {
      // Inserted within the middle of the queue.  Usually we don't have to wake
      // up the event queue unless there is a barrier at the head of the queue
      // and the message is the earliest asynchronous message in the queue.
      needWake = mBlocked && p.target == null && msg.isAsynchronous();
      Message prev;
      for (;;) {
        prev = p;
        p = p.next;
        if (p == null || when < p.when) {
          break;
        }
        if (needWake && p.isAsynchronous()) {
          needWake = false;
        }
      }
      msg.next = p; // invariant: p == prev.next
      prev.next = msg;
    }

    // We can assume mPtr != 0 because mQuitting is false.
    if (needWake) {
      nativeWake(mPtr);
    }
  }
  return true;
}
```

这个方法中主要就是将消息按照时间的先后顺序进行排队，通过 msg.next 指定下一个消息。

那么消息是如何出队的呢？这得看 Looper.loop() 源码了。

```java
public static void loop() {
  final Looper me = myLooper();
  if (me == null) {
    throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
  }
  final MessageQueue queue = me.mQueue;

  // Make sure the identity of this thread is that of the local process,
  // and keep track of what that identity token actually is.
  Binder.clearCallingIdentity();
  final long ident = Binder.clearCallingIdentity();

  for (;;) {
    Message msg = queue.next(); // might block
    if (msg == null) {
      // No message indicates that the message queue is quitting.
      return;
    }

    // This must be in a local variable, in case a UI event sets the logger
    final Printer logging = me.mLogging;
    if (logging != null) {
      logging.println(">>>>> Dispatching to " + msg.target + " " +
                      msg.callback + ": " + msg.what);
    }

    final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

    final long traceTag = me.mTraceTag;
    if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
      Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
    }
    final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
    final long end;
    try {
      msg.target.dispatchMessage(msg);  // 回调 Handler 的 dispatchMessage 方法
      end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
    } finally {
      if (traceTag != 0) {
        Trace.traceEnd(traceTag);
      }
    }
    if (slowDispatchThresholdMs > 0) {
      final long time = end - start;
      if (time > slowDispatchThresholdMs) {
        Slog.w(TAG, "Dispatch took " + time + "ms on "
               + Thread.currentThread().getName() + ", h=" +
               msg.target + " cb=" + msg.callback + " msg=" + msg.what);
      }
    }

    if (logging != null) {
      logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
    }

    // Make sure that during the course of dispatching the
    // identity of the thread wasn't corrupted.
    final long newIdent = Binder.clearCallingIdentity();
    if (ident != newIdent) {
      Log.wtf(TAG, "Thread identity changed from 0x"
              + Long.toHexString(ident) + " to 0x"
              + Long.toHexString(newIdent) + " while dispatching to "
              + msg.target.getClass().getName() + " "
              + msg.callback + " what=" + msg.what);
    }

    msg.recycleUnchecked();
  }
}
```

主线程中，这个方法在 ActivityThread 中的 main() 方法中被调用。可以看出，在这个方法中进入了一个死循环，不断调用 MessageQueue.next() 方法从消息队列中获取出队消息，将消息传递到 msg.target 的 dispatchMessage() 方法中，这里的 msg.target 其实就是 Handler，是在 Handler 的 enqueueMessage 方法中设置的。然后看一看 Handler 中 dispatchMessage() 方法的源码，如下所示：

```java
public void dispatchMessage(Message msg) {
  if (msg.callback != null) {
    handleCallback(msg);
  } else {
    if (mCallback != null) {
      if (mCallback.handleMessage(msg)) {
        return;
      }
    }
    handleMessage(msg);
  }
}
```

这里三种不同的调用方法都是对消息进行处理，提供不同的应用场景。

到这里 Android Handler 的基本原理一目了然了，对于其他一些在子线程中使用的方法，比如 Handler 的 post()方法 、Activity 的 runOnUiThread() 等方法，其背后的原理都是一样的，不再赘述。

## 总结

- 基本概念
  - Message：消息。
  - MessageQueue：消息队列，用来存放 Handler 发送过来的 Message，并且按照时间顺序将消息排成队列。
  - Handler：消息处理者，负责发送和处理消息。
  - Looper：消息轮询器，不断的从 MessageQqueue 中取出 Message 交给 Handler 处理。
- 应用启动时系统默认给 UI 主线程创建了 Looper 对象。如果是在子线程中创建 Handler 对象，需要先创建 Looper.prepare() 方法创建 Looper 对象。
- 一个线程只能拥有一个 Looper 对象和 一个 MessageQueue 对象。

