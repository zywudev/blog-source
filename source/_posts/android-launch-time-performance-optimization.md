---
title: Android 应用启动速度优化
date: 2019-09-09 10:55:36
tags: 
- 性能优化
- Android
---

应用启动时间的长短，影响到用户体验。对研发人员来说，启动速度是我们的“门面”。

本文主要分析如下几个问题：

- 应用启动有哪些流程？

- 如何检测应用启动耗时操作？

- 如何优化应用启动速度？

## 应用启动流程

应用的启动流程即从点击图标到用户可操作的全部过程。

启动分为三种类型：

- **冷启动**：当启动应用时，后台没有该应用的进程，这时系统会首先会创建一个新的进程分配给该应用。

- **热启动**：当启动应用时，后台已有该应用的进程，比如按下 home 键。

- **温启动**：当启动应用时，后台已有该应用的进程，但是启动的入口 Activity 被干掉了，比如按了 back 键，应用虽然退出了，但是该应用的进程是依然会保留在后台。

其中启动最慢的就是冷启动，系统和应用本身的工作都是从零开始。

冷启动开始时，系统有三个任务：

- 启动 App

- App 启动后显示一个空白的 Window

- 创建 App 的进程

在此之后，应用进程马上会执行以下任务：

- 创建 App 对象

- 启动 main thread

- 创建要启动的 Activity

- 加载 View

- 布置页面

- 进行第一次绘制

## 启动耗时监测

### adb shell am

使用 adb shell 获取应用启动时间：

```shell
adb shell am start -W [packageName]/[packageName.AppstartActivity]
```

输出的结果类似于：

```shell
$ adb shell am start -W com.speed.test/com.speed.test.HomeActivity
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.speed.test/.HomeActivity }
Status: ok
Activity: com.speed.test/.HomeActivity
ThisTime: 496    
TotalTime: 496
WaitTime: 503
Complete
```

- WaitTime 返回从 `startActivity` 到应用第一帧完全显示这段时间。 就是总的耗时，包括前一个应用 Activity `pause` 的时间和新应用启动的时间；

- ThisTime 表示一连串启动 Activity 的最后一个 Activity 的启动耗时；

- TotalTime 表示新应用启动的耗时，包括新进程的启动和 Activity 的启动，但不包括前一个应用 Activity `pause` 的耗时。

一般只需关注 TotalTime，即应用自身真正的启动耗时。

### TraceView

[TraceView](https://developer.android.google.cn/studio/profile/generate-trace-logs) 是 Android SDK 内置的一个工具，它可以加载 trace 文件，用图形的形式展示代码的执行时间、次数及调用栈，但工具本身带来的性能开销过大，有时无法反映真实的情况。比如一个函数本身的耗时是 1 秒，开启 TraceView 后可能会变成 5 秒。

TraceView 的具体使用可以看这篇文章 [Android 性能分析工具 TraceView](http://wuzhangyang.com/2019/09/04/android-traceview/)。

### Systrace + 函数插桩

Systrace 原理：在系统的一些关键链路（如SystemServcie、虚拟机、Binder驱动）插入一些信息（Label），
通过 Label 的开始和结束来确定某个核心过程的执行时间，然后把这些Label信息收集起来得到系统关键路径的运行时间信息，最后得到整个系统的运行性能信息。Android Framework 里面一些重要的模块都插入了 label 信息(Java 层通过 android.os.Trace 类完成，native层通过 ATrace 宏完成），用户 App 中可以添加自定义的 Lable，这样就组成了一个完成的性能分析系统。

具体使用教程可以看这篇文章：[手把手教你使用Systrace（一）](https://zhuanlan.zhihu.com/p/27331842)

Systrace 的优点：

- 可以看大整个流程系统和应用程序的调用流程。包括系统关键线程的函数调用，渲染耗时、线程锁、GC 耗时等。

- 性能损耗可以接受。

## 启动优化方案

### 闪屏主题

使用 Activity 的 windowBackground 属性为启动的 Activity 提供一个闪屏预览界面（layer-list），这样点击应用图标会立马显示闪屏界面。具体操作方法：

Layout XML：

```xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" android:opacity="opaque">
  <!-- 背景颜色 -->
  <item android:drawable="@android:color/white"/>
  <!-- 闪屏页图片 -->
  <item>
    <bitmap
      android:src="@drawable/product_logo_144dp"
      android:gravity="center"/>
  </item>
</layer-list>
```

style：

```xml
<style name="AppTheme.Launcher">
    <item name="android:windowFullscreen">true</item>
    <item name="android:windowBackground">@mipmap/layer-list</item>
</style>
```

Manifest：

```xml
<activity ...
android:theme="@style/AppTheme.Launcher" />
```

Activity：

```java
public class MyMainActivity extends AppCompatActivity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    // 替换回原来的主题，注意在 super.onCreate 之前调用
    setTheme(R.style.Theme_MyApp);
    super.onCreate(savedInstanceState);
    // ...
  }
}
```

这种方案在通过交互体验优化了展示效果，但并没有真正的加速启动。

对于中低端机，总的闪屏时间会更长，建议在 Android 6.0 或者 Android 7.0 以上才启用“闪屏优化” 方案。

### 异步初始化

一些可以异步初始化的任务，在不影响主线程加载的情况下，考虑放到工作线程去执行，避免阻塞主线程。

比如：Bugly、BlockCanary 等第三方组件初始化，数据库初始化工作。

设置工作线程优先级为 `THREAD_PRIORITY_BACKGROUND`，这样工作线程最多能获取到 10% 的时间片，优先保证主线程执行。

```java
public static void initThirdService() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                //设置线程的优先级，不与主线程抢资源
                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                initDb();
                initBugly()
                initBlockCanary();
                BiLogUtil.init(App.getInstance());
                ...
            }
        }).start();
    }
```

### 延迟初始化

首页渲染完成后，再初始化数据，也就是延迟初始化，目的就是让界面先显示出来，保证 UI 绘制的流畅性。

核心方法是在 Activity 的 `onCreate` 函数中加入下面的方法 ：

```java
getWindow().getDecorView().post(new Runnable() {
    @Override
    public void run() {
        myHandler.post(mLoadingRunnable);
    }
});
```

这里的 `run` 方法是在 Activity 的 `onResume` 之后执行的。

关于这种方案的机制参见 ：[Android 应用启动优化:一种 DelayLoad 的实现和原理(上篇)](https://www.androidperformance.com/2015/11/18/Android-app-lunch-optimize-delay-load/)

### 建议

1、减少布局层级，建议使用约束布局 ConstraintLayout。

2、去掉无用代码、重复逻辑。

3、避免 Application 中创建线程池，尽量延迟操作。

4、避免启动过多工作线程。

5、尽量减少 GC 的次数，避免造成主线程长时间的卡顿。

6、一句话准则：可以异步的都异步，不可以异步的尽量延迟。

## 参考

https://www.androidperformance.com/2015/11/18/Android-app-lunch-optimize-delay-load/

https://www.jianshu.com/p/f5514b1a826c

https://www.androidperformance.com/2015/11/18/Android-app-lunch-optimize-delay-load/

https://developer.android.google.cn/topic/performance/launch-time.html

https://zhuanlan.zhihu.com/p/27331842

http://wuzhangyang.com/2019/09/04/android-traceview/