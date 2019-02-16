---
title: Android 事件分发机制
date: 2018-10-10 17:13:11
tags: Android
---

## MotionEvent

根据面向对象思想，事件被封装成 MotionEvent 对象，以下是几个与手指触摸相关的常见事件:

- ACTION_DOWN : 手指初次触摸到屏幕时触发。
- ACTION_MOVE：手指在屏幕上滑动时触发，会多出触发。
- ACTION_UP：手指离开屏幕时触发。
- ACTION_CANCEL：事件被上层拦截时触发。

对于单指操作，一次触摸事件流程是这样的：

> 按下（ACTION_DOWN）--> 滑动（ACTION_MOVE）--> 离开（ACTION_UP)

如果只是简单的点击，则没有 ACTION_MOVE 事件产生。

## 事件分发、拦截与消费

与事件相关的三个重要方法：

- dispatchTouchEvent：事件分发机制中的核心，所有的事件调度都归它管。
- onInterceptTouchEvent：事件拦截。
- onTouchEvent：事件消费处理。

## 事件分发流程

**事件分发流程示意图**：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/FmgOuTi01vHo_79e1HMRnYZwG920.png)

> 图中 ViewGroup 与 View 之间省略了若干层 ViewGroup。

大致解释一下：

- 触摸事件都是先交由 Activity 的 `dispatchTouchEvent` 方法（在此之间还有一系列的操作，在此省略了），再一层层往下分发。当中间的 ViewGroup 不进行拦截时，事件会分发给最底层的 View，由 View 的 `onTouchEvent` 方法进行处理，如果事件一直未被处理，最后会返回到 Activity 的 `onTouchEvent`。
- 图中 View/ViewGroup 的 `onTouchEvent` 返回 false，并不是直接调用上层的 `onTouchEvent` 方法。而是上层的 `dispatchTouchEvent` 方法接收到下层的 false 返回值时，再将事件分发给自己的 `onTouchEvent` 处理。

- `onInterceptTouchEvent` 只存在于 ViewGroup 中。ViewGroup 是根据 `onInterceptTouchEvent` 的返回值来确定是调用子 View 的 `dispatchTouchEvent` 还是自身的 `onTouchEvent`， 并没有将调用交给 `onInterceptTouchEvent`。

## 源码分析

事件是从 Activity 开始分发，Activity 的 `dispatchTouchEvent` 是如何接受到触摸事件，还有一系列的前期工作，后面会单独写一篇文章描述。

### Activity 对事件的分发流程

#### Activity.dispatchTouchEvent

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        // 第一次按下时，用户希望与设备进行交互时，可以重写该方法实现
        onUserInteraction();
    }
    // 获取PhoneWindow, 传递事件
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    // 没有任何View处理事件时，交给Activity的onTouchEvent处理
    return onTouchEvent(ev);
}
```

其中 `getWindow` 返回的是 Activity 的 mWindow 成员变量，而 Window 类是一个抽象类，唯一实现类是 PhoneWindow，所以该方法获取到的是 PhoneWindow 对象。

#### getWindow().superDispatchTouchEvent

```java
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event); 
}
```

PhoneWindow 中直接将事件交给了 DecorView 处理，DecorView 的 `superDispatchTouchEvent` 方法如下。

```java
 public boolean superDispatchTouchEvent(MotionEvent event) {
     return super.dispatchTouchEvent(event);
 }
```

可以看到，DecorView 调用的是父类的 `dispatchTouchEvent` 方法，而 DecorView 的父类是 ViewGroup，所以接着会调用 `ViewGroup.dispatchTouchEvent`。

#### Activity.onTouchEvent

如果没有任何 view 处理事件，最后会交给 Activity 的 `onTouchEvent` 处理。

```java
 public boolean onTouchEvent(MotionEvent event) {
     // 当窗口需要关闭时，消费掉当前事件
     if (mWindow.shouldCloseOnTouch(this, event)) {
         finish();
         return true;
     }

     return false;
 }
```

### ViewGroup 对事件的分发流程

#### ViewGroup.dispatchTouchEvent

ViewGroup 的 `dispatchTouchEvent` 方法内容较多，这里先拆开来分析：

```java
// 检测拦截
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
    // 可以通过调用 requestDisallowInterceptTouchEvent,不让父 View 拦截事件
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {  // 是否允许调用拦截器
        intercepted = onInterceptTouchEvent(ev);  
        ev.setAction(action);
    } else {
        intercepted = false;
    }
} else {
    // 当前不是 ACTION_DOWN 事件，且没有触摸目标
    // 即子视图如果不消费 ACTION_DOWN，那么后续事件也不会分发到，当前父视图拦截处理
    intercepted = true;
}
```

这一段代码的目的是检测 ViewGroup 是否拦截事件。

mFirstTouchTarget 用来记录已经消费事件的子 View。

FLAG_DISALLOW_INTERCEPT 这个标志位可以影响到 ViewGroup 是否拦截事件，可以通过调用 `requestDisallowInterceptTouchEvent` 方法来设置，一般用于子 View 当中，禁止父 View 拦截事件，处理滑动冲突。但要注意，**`requestDisallowInterceptTouchEvent` 方法对 ACTION_DOWN 事件是无效的**，为什么呢？因为 **ViewGroup 的 `dispatchTouchEvent` 方法每次接收到 ACTION_DOWN 事件时，都会初始化状态**。代码如下：

```java
// 处理一个初始按下事件 ACTION_DOWN
if (actionMasked == MotionEvent.ACTION_DOWN) {
    // 发生 Action_DOWN 事件，取消清除之前所有的触摸目标
    cancelAndClearTouchTargets(ev);
    // 重置触摸状态，清除 FLAG_DISALLOW_INTERCEPT；设置 MFirstTouchTarget = null
    resetTouchState();
}
```

从上面分析我们知道 `onInterceptTouchEvent` 方法不是每次都会执行的。

如果 ViewGroup 没有拦截事件，事件会传递给它的子 View。

```java
// 不取消事件，同时不拦截事件, 并且是Down事件才进入该区域
if (!canceled && !intercepted) {
    // 寻找可以获取焦点的视图
    View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
        ? findChildWithAccessibilityFocus() : null;

    if (actionMasked == MotionEvent.ACTION_DOWN
        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
        final int actionIndex = ev.getActionIndex(); 
        final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
            : TouchTarget.ALL_POINTER_IDS;

        // 清空早先的触摸对象
        removePointersFromTouchTargets(idBitsToAssign);

        final int childrenCount = mChildrenCount;
        // 第一次 down 事件，同时子视图不为空时
        if (newTouchTarget == null && childrenCount != 0) {
            final float x = ev.getX(actionIndex);
            final float y = ev.getY(actionIndex);
            // 从视图最上层到最下层，获取所有能接收该事件的子视图
            final ArrayList<View> preorderedList = buildTouchDispatchChildList();
            final boolean customOrder = preorderedList == null
                && isChildrenDrawingOrderEnabled();
            final View[] children = mChildren;
            // 从最底层的父视图开始遍历
            for (int i = childrenCount - 1; i >= 0; i--) {
                final int childIndex = getAndVerifyPreorderedIndex(
                    childrenCount, i, customOrder);
                final View child = getAndVerifyPreorderedView(
                    preorderedList, children, childIndex);

                // 如果子view无法获取用户焦点，跳过本次循环
                if (childWithAccessibilityFocus != null) {
                    if (childWithAccessibilityFocus != child) {
                        continue;
                    }
                    childWithAccessibilityFocus = null;
                    i = childrenCount - 1;
                }
                // 子view可见，并且没有播放动画；触摸事件的坐标落在子view的范围内
                // 如果这两个条件有一项不满足，则跳过此次循环
                if (!canViewReceivePointerEvents(child)
                    || !isTransformedTouchPointInView(x, y, child, null)) {
                    ev.setTargetAccessibilityFocus(false);
                    continue;
                }

                newTouchTarget = getTouchTarget(child);
                // 子 view 已经接受触摸事件，将新指针id赋值给它，退出循环
                if (newTouchTarget != null) {
                    newTouchTarget.pointerIdBits |= idBitsToAssign;
                    break;
                }

                resetCancelNextUpFlag(child);
                // 如果触摸位置在子view的区域内，把事件分发给子view或者ViewGroup
                // 实际调用的是chlid的dispatchTouchEvent方法
                if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                    // 子view消费了事件
                    mLastTouchDownTime = ev.getDownTime();
                    if (preorderedList != null) {
                        // childIndex points into presorted list, find original index
                        for (int j = 0; j < childrenCount; j++) {
                            if (children[childIndex] == mChildren[j]) {
                                mLastTouchDownIndex = j;
                                break;
                            }
                        }
                    } else {
                        mLastTouchDownIndex = childIndex;
                    }
                    mLastTouchDownX = ev.getX();
                    mLastTouchDownY = ev.getY();
                    // 设置接收事件的子view为新的触摸目标，设置为触摸目标链表的头
                    newTouchTarget = addTouchTarget(child, idBitsToAssign);
                    alreadyDispatchedToNewTouchTarget = true;
                    // 子view处理了事件，就跳出for循环
                    break;
                }

                // The accessibility focus didn't handle the event, so clear
                // the flag and do a normal dispatch to all children.
                ev.setTargetAccessibilityFocus(false);
            }
            if (preorderedList != null) preorderedList.clear();
        }

        if (newTouchTarget == null && mFirstTouchTarget != null) {
            // Did not find a child to receive the event.
            // Assign the pointer to the least recently added target.
            newTouchTarget = mFirstTouchTarget;
            while (newTouchTarget.next != null) {
                newTouchTarget = newTouchTarget.next;
            }
            newTouchTarget.pointerIdBits |= idBitsToAssign;
        }
    }
}
```

当事件没有被取消，没有被拦截，并且是 ACTION_DOWN 事件时，才会进入上述代码区域，寻找子 View 进行事件分发。

从代码中可以看到，ViewGroup 从最底层开始遍历，找寻 newTouchTarget。如果已经存在找寻newTouchTarget，说明正在接收触摸事件，则跳出循环。









































4**、onInterceptTouchEvent 返回 true 表示事件拦截，onTouchEvent 返回 true 表示事件消费。

**5**、事件在从 Activity.dispatchTouchEvent 往下分发的过程中。

如果中间的 ViewGroup 都不拦截，进入最底层的 View 后，由View.onTouchEvent 处理，如果 View 也没有消费事件，最后会返回到 Activity.onTouchEvent。

如果中间任何一层 ViewGroup 拦截事件，则事件不再往下分发，交由拦截的 ViewGroup 的 onTouchEvent 来处理。

**6**、如果 View 没有消费 ACTION_DOWN 事件，则之后的 ACTION_MOVE 等事件都不会再接收。



## 参考

- [Android事件分发机制](http://gityuan.com/2015/09/19/android-touch/)
- [事件分发机制原理](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B12%5DDispatch-TouchEvent-Theory.md)