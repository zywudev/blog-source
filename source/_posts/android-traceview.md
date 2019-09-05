---
title: Android 性能分析工具 TraceView
date: 2019-09-04 10:48:52
tags:
- 性能优化
- Android
---

在做应用启动、卡顿优化时，经常会用到 Android 性能分析工具 TraceView，这里简单介绍下 TraceView 的基础使用。

### TraceView 是什么

[TraceView](https://developer.android.google.cn/studio/profile/generate-trace-logs) 是 Android SDK 内置的一个工具，它可以加载 **trace** 文件，用图形的形式展示**代码的执行时间、次数及调用栈**，便于我们分析。

### 生成 trace 文件

trace 文件是 log 信息文件的一种，可以通过代码，Android Studio，或者 DDMS 生成。

1\. 使用代码生成 trace 文件

在想要记录的地方调用 `Debug.startMethodTracing("sample")`，参数指定 `trace` 文件的名称。

在结束记录的地方调用 `Debug.stopMethodTracing()`，文件会被保存到 `/sdcard/Android/data/packageName/files` 文件夹下，

```java
Debug.startMethodTracing("sample"); // 开始 trace
...
Debug.stopMethodTracing();  // 结束 trace
```



