---
title: Android 事件分发机制
date: 2018-10-10 17:13:11
tags: Android
---

## MotionEvent

根据面向对象思想，事件被封装成 MotionEvent 对象，以下是几个与手指触摸相关的常见事件:

- ACTION_DOWN : 手指初次触摸到屏幕时触发。
- ACTION_MOVE：手指在屏幕上滑动时触发，会触发多次。
- ACTION_UP：手指离开屏幕时触发。
- ACTION_CANCEL：事件被上层拦截时触发。

对于单指操作，一次触摸事件流程是这样的：

按下（ACTION_DOWN）--> 滑动（ACTION_MOVE）--> 离开（ACTION_UP)。如果只是简单的点击，则没有 ACTION_MOVE 事件产生。

## 事件分发、拦截与消费

与事件分发相关的三个重要方法：

- dispatchTouchEvent：事件分发机制中的核心，所有的事件调度都归它管。
- onInterceptTouchEvent：事件拦截。
- onTouchEvent：事件消费处理。

## 事件分发流程

事件分发流程示意图：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/FmgOuTi01vHo_79e1HMRnYZwG920.png)

大致解释一下：

- 图中 ViewGroup 与 View 之间省略了若干层 ViewGroup。

- 触摸事件都是先交由 Activity 的 `dispatchTouchEvent` 方法（在此之间还有一系列的操作，在此省略了），再一层层往下分发。当中间的 ViewGroup 不进行拦截时，事件会分发给最底层的 View，由 View 的 `onTouchEvent` 方法进行处理，如果事件一直未被处理，最后会返回到 Activity 的 `onTouchEvent`。
- 图中 View/ViewGroup 的 `onTouchEvent` 返回 false，并不是直接调用上层的 `onTouchEvent` 方法。而是上层的 `dispatchTouchEvent` 方法接收到下层的 false 返回值时，再将事件分发给自己的 `onTouchEvent` 处理。

- `onInterceptTouchEvent` 只存在于 ViewGroup 中。ViewGroup 是根据 `onInterceptTouchEvent` 的返回值来确定是调用子 View 的 `dispatchTouchEvent` 还是自身的 `onTouchEvent`， 并没有将调用交给 `onInterceptTouchEvent`。

## 源码分析

事件是从 Activity 开始分发，Activity 的 `dispatchTouchEvent` 是如何接受到触摸事件，还有一系列的前期工作，后面会单独写一篇文章总结。

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

可以看到，PhoneWindow 中直接将事件交给了 DecorView 处理，DecorView 的 `superDispatchTouchEvent` 方法如下。

```java
 public boolean superDispatchTouchEvent(MotionEvent event) {
     return super.dispatchTouchEvent(event);
 }
```

DecorView 调用的是父类的 `dispatchTouchEvent` 方法，而 DecorView 的父类是 ViewGroup，所以接着会调用 `ViewGroup.dispatchTouchEvent`。

#### Activity.onTouchEvent

如果没有任何 View 处理事件，最后会交给 Activity 的 `onTouchEvent` 处理。

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

ViewGroup 的 `dispatchTouchEvent` 方法内容较多，这里拆分为检测拦截、寻找子 View、分发事件。

**1、 检测拦截**

```java
// 检测拦截
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
    // 可以通过调用 requestDisallowInterceptTouchEvent,不让父 View 拦截事件
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {  // 允许父 View 拦截
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

mFirstTouchTarget 用来记录已经消费事件的子 View，目的是为了后续其他事件分发时直接将事件分发给 mFirstTouchTarget  指向的 View。

`FLAG_DISALLOW_INTERCEPT` 这个标志位可以影响到 ViewGroup 是否拦截事件，可以通过调用 `requestDisallowInterceptTouchEvent` 方法来设置，一般用于子 View 当中，禁止父 View 拦截事件，处理滑动冲突。但要注意，**`requestDisallowInterceptTouchEvent` 方法对 ACTION_DOWN 事件是无效的**，为什么呢？因为 **ViewGroup 的 `dispatchTouchEvent` 方法每次接收到 ACTION_DOWN 事件时，都会初始化状态**。代码如下：

```java
// 处理一个初始按下事件 ACTION_DOWN
if (actionMasked == MotionEvent.ACTION_DOWN) {
    // 发生 Action_DOWN 事件，取消清除之前所有的触摸目标
    cancelAndClearTouchTargets(ev);
    // 重置触摸状态，清除 FLAG_DISALLOW_INTERCEPT；设置 mFirstTouchTarget = null
    resetTouchState();
}
```

**2、寻找子 View**

如果 ViewGroup 没有拦截事件，事件没有被取消，并且是 ACTION_DOWN 事件时，首先会去寻找可以接收事件的子 View。代码如下：

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

                ev.setTargetAccessibilityFocus(false);
            }
            if (preorderedList != null) preorderedList.clear();
        }

        if (newTouchTarget == null && mFirstTouchTarget != null) {
            // 将 mFirstTouchTarget 的链表最后的 TouchTarget 赋给 newToughTarget
            newTouchTarget = mFirstTouchTarget;
            while (newTouchTarget.next != null) {
                newTouchTarget = newTouchTarget.next;
            }
            newTouchTarget.pointerIdBits |= idBitsToAssign;
        }
    }
}
```

寻找子 View 进行分发事件的方法就是遍历子 View，有这样两个条件：

```java
// 1、子view可见，并且没有播放动画；2、触摸事件的坐标落在子view的范围内
if (!canViewReceivePointerEvents(child)
    || !isTransformedTouchPointInView(x, y, child, null)) {
    ev.setTargetAccessibilityFocus(false);
    continue;
}
```

这两个条件如果同时满足，则将事件分发给子 View。

接着会调用 `dispatchTransformedTouchEvent` 方法，可以猜到这个方法中肯定做了事件分发的操作。

如果这个方法返回 true，表示子 View 消费了事件，则会在 `addTouchTarget` 方法中设置 mFirstTouchTarget ，后续事件（ACTION_MOVE、ACTION_UP）分发时会直接将事件分发给 mFirstTouchTarget 指向的 View。

换句话说，如果子 View 没有消费 ACTION_DOWN 事件，mFirstTouchTarget 就会为 null，也就不会接收其他 ACTION_MOVE、ACTION_UP 等事件，如下代码：

```java
if (mFirstTouchTarget == null) {
    // 没有触摸target,则由当前ViewGroup来处理（第三个参数child为null）
    handled = dispatchTransformedTouchEvent(ev, canceled, null,
                                            TouchTarget.ALL_POINTER_IDS);
} else {
    // 如果View消费ACTION_DOWN事件，那么ACTION_MOVE、ACTION_UP等事件相继开始执行
    TouchTarget predecessor = null;
    TouchTarget target = mFirstTouchTarget;
    while (target != null) {
        final TouchTarget next = target.next;
        if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
            handled = true;
        } else {
            final boolean cancelChild = resetCancelNextUpFlag(target.child)
                || intercepted;
            if (dispatchTransformedTouchEvent(ev, cancelChild,
                                              target.child, target.pointerIdBits)) {
                handled = true;
            }
            if (cancelChild) {
                if (predecessor == null) {
                    mFirstTouchTarget = next;
                } else {
                    predecessor.next = next;
                }
                target.recycle();
                target = next;
                continue;
            }
        }
        predecessor = target;
        target = next;
    }
}
```

**3、事件分发**

`dispatchTransformedTouchEvent` 的伪代码如下：

```java
 if (child == null) {
     handled = super.dispatchTouchEvent(event);  
 } else {
     handled = child.dispatchTouchEvent(event);
 }
```

如果不存在子 View，ViewGroup 会调用父类 View 的 `dispatchTouchEvent` 方法，再调用 onTouchEvent 方法处理事件。

如果存在子 View ,将事件分发给子 View 的 `dispatchTouchEvent`，子 View 如果是 ViewGroup，则会调用 `ViewGroup.dispatchTouchEvent`，进行拦截检测、寻找子 View、分发事件操作。

### View 对事件的分发流程

#### View.dispatchTouchEvent

```java
public boolean dispatchTouchEvent(MotionEvent event) {
    ...
    
    final int actionMasked = event.getActionMasked();
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // 在ACION_DOWN事件之前，如果存在滚动操作则停止。不存在则不进行操作
        stopNestedScroll();
    }

    if (onFilterTouchEventForSecurity(event)) {
        // 当存在OnTouchListener，且视图状态为ENABLED时，调用onTouch方法
        // mOnTouchListener.onTouch优先于onTouchEvent执行
        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnTouchListener != null
                && (mViewFlags & ENABLED_MASK) == ENABLED
                && li.mOnTouchListener.onTouch(this, event)) {
            result = true; // 如果已经消费事件，则返回True
        }
        //如果OnTouch没有消费Touch事件则调用OnTouchEvent
        if (!result && onTouchEvent(event)) { 
            result = true; //如果已经消费事件，则返回True
        }
    }

    if (!result && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
    }

    // 处理取消或抬起操作
    if (actionMasked == MotionEvent.ACTION_UP ||
            actionMasked == MotionEvent.ACTION_CANCEL ||
            (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
        stopNestedScroll();
    }

    return result;
}
```

如果存在 OnTouchListener，且视图状态为 ENABLED 时，调用`onTouch` 方法，`onTouch` 方法会优先处理事件。

如果 `onTouch` 方法返回 true，表示已经消费了事件，也就不再执行 `onTouchEvent` 。否则， `onTouchEvent` 处理事件，返回 true，消费事件，否则不处理事件。

#### View.onTouchEvent

```java
public boolean onTouchEvent(MotionEvent event) {
    final float x = event.getX();
    final float y = event.getY();
    final int viewFlags = mViewFlags;

    // 当View状态为DISABLED，如果可点击或可长按，则返回True，即消费事件
    if ((viewFlags & ENABLED_MASK) == DISABLED) {
        if (event.getAction() == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);
        }
        return (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
    }

    if (mTouchDelegate != null) {
        if (mTouchDelegate.onTouchEvent(event)) {
            return true;
        }
    }

    //当View状态为ENABLED，如果可点击或可长按，则返回True，即消费事件;
    //与前面的的结合，可得出结论:只要view是可点击或可长按，则消费该事件.
    if (((viewFlags & CLICKABLE) == CLICKABLE ||
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_UP:
                boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                    boolean focusTaken = false;
                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                        focusTaken = requestFocus();
                    }

                    if (prepressed) {
                        setPressed(true, x, y);
                   }

                    if (!mHasPerformedLongPress) {
                        //这是Tap操作，移除长按回调方法
                        removeLongPressCallback();

                        if (!focusTaken) {
                            if (mPerformClick == null) {
                                mPerformClick = new PerformClick();
                            }
                            //调用View.OnClickListener
                            if (!post(mPerformClick)) {
                                performClick();
                            }
                        }
                    }

                    if (mUnsetPressedState == null) {
                        mUnsetPressedState = new UnsetPressedState();
                    }

                    if (prepressed) {
                        postDelayed(mUnsetPressedState,
                                ViewConfiguration.getPressedStateDuration());
                    } else if (!post(mUnsetPressedState)) {
                        mUnsetPressedState.run();
                    }

                    removeTapCallback();
                }
                break;

            case MotionEvent.ACTION_DOWN:
                mHasPerformedLongPress = false;

                if (performButtonActionOnTouchDown(event)) {
                    break;
                }

                //获取是否处于可滚动的视图内
                boolean isInScrollingContainer = isInScrollingContainer();

                if (isInScrollingContainer) {
                    mPrivateFlags |= PFLAG_PREPRESSED;
                    if (mPendingCheckForTap == null) {
                        mPendingCheckForTap = new CheckForTap();
                    }
                    mPendingCheckForTap.x = event.getX();
                    mPendingCheckForTap.y = event.getY();
                    //当处于可滚动视图内，则延迟TAP_TIMEOUT，再反馈按压状态
                    // 用来判断用户是否想要滚动。默认延时为100ms
                    postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                } else {
                    //当不再滚动视图内，则立刻反馈按压状态
                    setPressed(true, x, y);
                    checkForLongClick(0); //检测是否是长按
                }
                break;

            case MotionEvent.ACTION_CANCEL:
                setPressed(false);
                removeTapCallback();
                removeLongPressCallback();
                break;

            case MotionEvent.ACTION_MOVE:
                drawableHotspotChanged(x, y);

                if (!pointInView(x, y, mTouchSlop)) {
                    removeTapCallback();
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                        removeLongPressCallback();
                        setPressed(false);
                    }
                }
                break;
        }

        return true;
    }
    return false;
}
```

只要 view 是可点击或可长按，则消费该事件。

长按事件是在 ACTION_DOWN 事件中检测，单击事件需要两个事件 ACTION_DOWN、ACTION_UP 才能触发。

与 View 相关的各个方法调用顺序应该是这样的：

onTouchListener > onTouchEvent > onLongClickListener > onClickListener

## 总结

结合源码和事件分发示意图，对事件分发机制总结一下：

1、事件在从 Activity 的 `dispatchTouchEvent` 往下分发，如果没有 View 消费事件，事件最后会回到 Activity 的 `onTouchEvent` 方法处理。

2、ViewGroup 可以对事件进行拦截，拦截后事件不再往子 View 分发，交由发生拦截操作的 ViewGroup 的 `onTouchEvent` 处理。

3、子 View 可以调用 `requestDisallowInterceptTouchEvent` 方法进行设置，从而阻止父 ViewGroup 的 `onInterceptTouchEvent` 拦截事件。

4、如果 View 没有消费 ACTION_DOWN 事件，则之后的 ACTION_MOVE 等事件都不会再接收。

5、只要 View 是可点击或者可长按的，则消费该事件。

6、如果当前正在处理的事件被上层拦截，会收到一个 ACTION_CANCEL，后续事件不会再传递过来。

## 参考

- [Android事件分发机制](http://gityuan.com/2015/09/19/android-touch/)
- [事件分发机制原理](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B12%5DDispatch-TouchEvent-Theory.md)