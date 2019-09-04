---
title: Android 性能分析工具 TraceView
date: 2019-09-04 10:48:52
tags:
- 性能优化
- Android
---

在做应用启动、卡顿优化时，经常会用到 Android 性能分析工具 TraceView，这里简单介绍下 TraceView 的基础使用。

## TraceView 是什么

[TraceView](https://developer.android.google.cn/studio/profile/generate-trace-logs) 是 Android SDK 内置的一个工具，它利用 Android Runtime 函数调用的 event 事件，将函数运行的耗时和调用关系写入 trace 文件中。

## TraceView 使用方式

在想要记录的地方调用 `Debug.startMethodTracing("sample")`，参数指定 `trace` 文件的名称。

在结束记录的地方调用 `Debug.stopMethodTracing()`，文件会被保存到 `/sdcard/Android/data/packageName/files` 文件夹下。

```java
Debug.startMethodTracing("sample");
...
Debug.stopMethodTracing();
```

Android Studio 的  Profiler 可以直接打开 trace 文件。



