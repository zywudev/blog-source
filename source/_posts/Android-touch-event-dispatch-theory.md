---
title: Android 事件分发原理
date: 2018-10-10 17:13:11
tags:
---

## 事件分发流程图

![](https://github.com/zywudev/blog-source/blob/master/image/FmgOuTi01vHo_79e1HMRnYZwG920.png)

**1**、当 UI 主线程收到触摸 input 事件，经过一系列处理，最终会走到 DecorView 的 dispatchTouchEvent 方法。

```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    final Window.Callback cb = mWindow.getCallback();
    return cb != null && !mWindow.isDestroyed() && mFeatureId < 0
        ? cb.dispatchTouchEvent(ev) : super.dispatchTouchEvent(ev);
}
```

Activity 实现了 Window.Callback 接口，所以接下来会调用 Activity 的 dispatchTouchEvent，所以可以将 Activity 作为原始的事件分发者。

**2**、事件分发、拦截与消费

| 类型     | 相关方法              | Activity | ViewGroup | View |
| -------- | --------------------- | -------- | --------- | ---- |
| 事件分发 | dispatchTouchEvent    | √        | √         | √    |
| 事件拦截 | onInterceptTouchEvent | X        | √         | X    |
| 事件消费 | onTouchEvent          | √        | √         | √    |

其中，Activity 与 View 没有事件拦截，主要原因是：

Activity 作为原始的事件分发者，如果 Activity 拦截了事件会导致整个屏幕无法响应事件，不是我们想要的效果；View 作为事件传递的最末端，要么消费事件，要么不处理事件进行回传，根本没必要拦截。

**3**、事件分发流程

Android View 是树形结构，事件分发流程采用的是责任链模式。

事件传递：

```
Activity －> PhoneWindow －> DecorView －> ViewGroup －> ... －> View
```

事件回传：

```
Activity <－ PhoneWindow <－ DecorView <－ ViewGroup <－ ... <－ View
```

**4**、onInterceptTouchEvent 返回 true 表示事件拦截，onTouchEvent 返回 true 表示事件消费，

**5**、事件在从 Activity.dispatchTouchEvent 往下分发的过程中。

如果中间的 ViewGroup 都不拦截，进入最底层的 View 后，由View.onTouchEvent 处理，如果 View 也没有消费事件，最后会返回到 Activity.onTouchEvent。

如果中间任何一层 ViewGroup 拦截事件，则事件不再往下分发，交由拦截的 ViewGroup 的 onTouchEvent 来处理。

**6**、如果 View 没有消费 ACTION_DOWN 事件，则之后的 ACTION_MOVE 等事件都不会再接收。

## 参考

- [Android事件分发机制](http://gityuan.com/2015/09/19/android-touch/)
- [事件分发机制原理](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B12%5DDispatch-TouchEvent-Theory.md)