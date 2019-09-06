---
title: Android 性能分析工具 TraceView
date: 2019-09-04 10:48:52
tags:
- 性能优化
- Android
---

在做应用启动、卡顿优化时，经常会用到 Android 性能分析工具 TraceView，这里简单介绍下 TraceView 的基础使用。

## TraceView 是什么

[TraceView](https://developer.android.google.cn/studio/profile/generate-trace-logs) 是 Android SDK 内置的一个工具，它可以加载 **trace** 文件，用图形的形式展示**代码的执行时间、次数及调用栈**，便于我们分析。

## 生成 trace 文件

trace 文件是 log 信息文件的一种，可以通过代码，Android Studio，或者 DDMS 生成。

### 使用代码生成 trace 文件

在想要记录的地方调用 `Debug.startMethodTracing("sample")`，参数指定 `trace` 文件的名称。

在结束记录的地方调用 `Debug.stopMethodTracing()`，文件会被保存到 `/sdcard/Android/data/packageName/files` 文件夹下。

```java
Debug.startMethodTracing("sample"); // 开始 trace
...
Debug.stopMethodTracing();  // 结束 trace
```

可以使用 adb 命令导出 trace 文件，使用 Android Studio Profiler 或者 DDMS 打开。

### 使用 Android Studio 生成 trace 文件

点击工具栏中的 Profiler（Android Studio 版本是 3.4.2）, 点击 CPU 时间轴上的任意位置以打开 CPU Profiler。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/cpu_profiler1.png)

1\.  **事件时间轴**：显示应用中的 Activity 在其生命周期内不断转换而经历各种不同状态的过程，并指示用户与设备的交互，包括屏幕旋转事件。

2\. **CPU 时间轴** : 显示应用的实时 CPU 使用率以及应用当前使用的线程总数。通过沿时间线的水平轴移动鼠标，还可以检查历史 CPU 使用率数据。

3\. **线程活动时间轴**：应用进程的所有线程。不同颜色对应的含义：

- **绿色**： 表示线程处于活动状态或准备使用 CPU。 即，它正在“运行中”或处于“可运行”状态。

- **黄色**： 表示线程处于活动状态，但它正在等待一个 I/O 操作（如磁盘或网络 I/O），然后才能完成它的工作。

- **灰色**： 表示线程正在休眠且没有消耗任何 CPU 时间。 当线程需要访问尚不可用的资源时偶尔会发生这种情况。 线程进入自主休眠或内核将此线程置于休眠状态，直到所需的资源可用。

要开始记录跟踪数据，点击 CPU Profiler 顶部的下拉框选择适当的记录配置：

- **对 Java 方法采样**：在应用的 Java 代码执行期间，频繁捕获应用的调用堆栈。分析器会比较捕获的数据集，以推导与应用的 Java 代码执行有关的时间和资源使用信息。

- **跟踪 Java 方法** ：在运行时检测应用，以在每个方法调用开始和结束时记录一个时间戳。系统会收集并比较这些时间戳，以生成方法跟踪数据，包括时间信息和 CPU 使用率。

- **对 C/C++ 函数采样**：捕获应用的原生线程的采样跟踪数据。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/cpu_profiler2.png)

选择配置后，点击 `Record` 进行跟踪，交互完成后点击 `Stop` 结束数据跟踪。分析器会分析 trace 数据，如下图所示。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/cpu_profiler3.png)

1\. **选择时间范围**： 确定要在跟踪窗格中检查所记录时间范围的哪一部分。 当首次记录函数跟踪时，CPU Profiler 将在 CPU 时间线中自动选择完整长度。 如果想仅检查所记录时间范围一小部分的函数跟踪数据，可以点击并拖动突出显示的区域边缘以修改其长度。

2\. **时间戳**： 用于表示所记录函数跟踪的开始和结束时间（相对于分析器从设备开始收集 CPU 使用率信息的时间）。 可以点击时间戳以自动选择完整记录。

3\. **跟踪窗格**： 用于显示所选的时间范围和线程的函数跟踪数据。

4\. **跟踪数据窗格标签**：通过Call Chart（调用图表）、Flame Chart（火焰图）、 Top Down 树或 Bottom Up 树的形式显示函数跟踪。

- **Call Chart** : 水平轴表示函数调用（或调用方）的时间，并沿垂直轴显示其被调用者。 对系统 API 的函数调用显示为橙色，对应用自有函数的调用显示为绿色，对第三方 API（包括 Java 语言 API）的函数调用显示为蓝色。 

- **Flame Chart**: 一个倒置的调用图表，其中水平轴不再代表时间线，它表示每个函数相对的执行时间。

- **Top Down**：显示一个函数调用列表，在该列表中展开函数节点会显示函数的被调用方。

- **Bottom Up**：显示一个函数调用列表，在该列表中展开函数节点将显示函数的调用方。

5\. **时间参考菜单** ：确定如何测量每个函数调用的时间信息：

- **Wall clock time**：实际经过的时间。

- **Thread time**：实际经过的时间减去线程没有消耗 CPU 资源的时间。

### 使用 DDMS 生成 trace 文件

DDMS 即 Dalvik Debug Monitor Server ，是 Android 调试监控工具，它为我们提供了截图，查看 log，查看视图层级，查看内存使用等功能。

Android Studio 3.0 后可在 Android SDK 的 `tools` 目录，找到 `monitor.bat`，使用命令行启动它，就能打开 DDMS。 

DDMS 界面点击 `Start Method Profiling` 按钮，开始记录 trace，同一个按钮停止 trace。DDMS 会自动启用 TraceView 加载 trace 文件，如下图：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/ddms_trace.png)

图中上半部分展示了不同线程的执行时间，其中不同的颜色代表不同的方法，同一颜色越长，说明执行时间越长，空白表示这个时间段内没有执行内容。

下半部分展示了不同方法的执行时间信息。各个指标的含义：

- Incl Cpu Time：方法占用的 CPU 时间（包括调用子函数所消耗的时间）。

- Excl Cpu Time：方法自身占用的 CPU 时间（不包括调用其他方法所消耗的时间）。

- Incl Real Time：方法运行的真实时间（包括调用子函数所消耗的时间）。

- Excl Real Time：方法自身运行的真实时间（不包括调用其他方法所消耗的时间）。

- Calls+RecurCalls/Total：方法被调用的次数+重复调用的次数。

- Cpu Time/Call：方法调用 CPU 时间与调用次数的比，相当于方法平均执行时间。

- Real Time/Call：同 Cpu Time/Call 类似，只不过统计单位换成了真实时间。

在分析耗时的时候一般有两种情况：

- 调用次数不多。但是，本身就非常耗时。

- 本身不是很耗时。但是，调用非常频繁。

第一种情况，可以使用 `Cpu Time` 来查看它的耗时情况。 

第二种情况，可以使用 `Calls+RecurCalls/Total` 来查看它的调用情况。

## 参考文件

- https://developer.android.google.cn/studio/profile/cpu-profiler
- https://blog.csdn.net/u011240877/article/details/54347396