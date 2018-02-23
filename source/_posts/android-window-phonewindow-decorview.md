---
title: Android setContentView，Window，PhoneWindow and DecorView 
date: 2018-02-23 10:03:06
tags:
---

从 Activity 的 `setContentView(R.layout.activity_main)` 开始。

```java
public void setContentView(@LayoutRes int layoutResID) {
  getWindow().setContentView(layoutResID);
  initWindowDecorActionBar();
}
```

这里调用了 `getWindow().setContentView` 方法，那这个 getWindow() 是什么呢？

```java
/**
     * Retrieve the current {@link android.view.Window} for the activity.
     * This can be used to directly access parts of the Window API that
     * are not available through Activity/Screen.
     *
     * @return Window The current window, or null if the activity is not
     *         visual.
     */
public Window getWindow() {
  return mWindow;
}
```

从上面的 getWindow() 方法可以看到，获取的是当前 activity 的 Window 对象，如果 activity 不可见，返回 null。而 Window 是一个抽象类，所以这里的 mWindow.setContentView 方法调用的是 Window 实现类的方法。那 Window 的实现类是什么呢？

在 Activity 类的 attach 方法中，可以看出，PhoneWindow 即是 Window 的实现类。

```java
final void attach(Context context, ActivityThread aThread,
                  Instrumentation instr, IBinder token, int ident,
                  Application application, Intent intent, ActivityInfo info,
                  CharSequence title, Activity parent, String id,
                  NonConfigurationInstances lastNonConfigurationInstances,
                  Configuration config, String referrer, IVoiceInteractor voiceInteractor,
                  Window window, ActivityConfigCallback activityConfigCallback) {
  attachBaseContext(context);

  mFragments.attachHost(null /*parent*/);
  // 创建 PhoneWindow 对象
  mWindow = new PhoneWindow(this, window, activityConfigCallback);
  ... // 省略
}
```

所以我们直接去看 PhoneWindow 的 setContentView 方法。

```java
public void setContentView(int layoutResID) {
  // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
  // decor, when theme attributes and the like are crystalized. Do not check the feature
  // before this happens.
  if (mContentParent == null) {
    installDecor(); 
  } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
    mContentParent.removeAllViews();
  }
   ... // 省略
}
```

如果 mContentParent == null，也就是当前内容未加载到窗口中，第一次加载时，调用 installDecor() 方法。

```java
 private void installDecor() {
   mForceDecorInstall = false;
   if (mDecor == null) {
     mDecor = generateDecor(-1);  
     mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
     mDecor.setIsRootNamespace(true);
     if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
       mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
     }
   } else {
     mDecor.setWindow(this);
   }
   if (mContentParent == null) {
     mContentParent = generateLayout(mDecor);
     ... // 省略 
   }
 }
```

installDecor() 方法中先是通过 generateDecor() 方法创建 DecorView 对象，DecorView 继承于 FrameLayout，用来作为整个PhoneWindow的根视图，但此时还只是个空架子。

```java
protected DecorView generateDecor(int featureId) {
  ... // 省略
    return new DecorView(context, featureId, this, getAttributes());
}
```

创建完成后，根据用户的设置，通过 generateLayout() 方法根据 Theme 从系统中选择创建默认布局。

**引用 [事件分发机制原理](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B12%5DDispatch-TouchEvent-Theory.md) 中的例子阐述 Window、PhoneWindow 及 DecorView 之间的关系**：

> 简单来说，Window是一个抽象类，是所有视图的最顶层容器，视图的外观和行为都归他管，不论是背景显示，标题栏还是事件处理都是他管理的范畴，它其实就像是View界的太上皇(虽然能管的事情看似很多，但是没实权，因为抽象类不能直接使用)。
>
> 而 PhoneWindow 作为 Window 的唯一亲儿子(唯一实现类)，自然就是 View 界的皇帝了，PhoneWindow 的权利可是非常大大，不过对于我们来说用处并不大，因为皇帝平时都是躲在深宫里面的，虽然偶尔用特殊方法能见上一面，但想要完全指挥 PhoneWindow 为你工作是很困难的。
>
> 而上面说的 DecorView 是 PhoneWindow 的一个内部类，其职位相当于小太监，就是跟在 PhoneWindow 身边专业为 PhoneWindow 服务的，除了自己要干活之外，也负责消息的传递，PhoneWindow 的指示通过 DecorView 传递给下面的 View，而下面 View 的信息也通过 DecorView 回传给 PhoneWindow。