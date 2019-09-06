---
title: Android 应用启动速度优化方案
date: 2019-09-03 10:55:36
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

在此之后，应用进程马上回执行以下任务：

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

- WaitTime 返回从 `startActivity` 到应用第一帧完全显示这段时间. 就是总的耗时，包括前一个应用 Activity `pause` 的时间和新应用启动的时间；

- ThisTime 表示一连串启动 Activity 的最后一个 Activity 的启动耗时；

- TotalTime 表示新应用启动的耗时，包括新进程的启动和 Activity 的启动，但不包括前一个应用 Activity `pause` 的耗时。

一般只需关注 TotalTime，即应用自身真正的启动耗时。

### TraceView

[TraceView](https://developer.android.google.cn/studio/profile/generate-trace-logs) 是 Android SDK 内置的一个工具，它可以加载 trace 文件，用图形的形式展示代码的执行时间、次数及调用栈，但工具本身带来的性能开销过大，有时无法反映真实的情况。比如一个函数本身的耗时是 1 秒，开启 TraceView 后可能会变成 5 秒。

TraceView 的具体使用可以看这篇文章 [Android 性能分析工具 TraceView](http://wuzhangyang.com/2019/09/04/android-traceview/)。

### Systrace 





