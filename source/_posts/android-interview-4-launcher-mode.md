---
title: Android 面试题（4）：谈谈 Activity 的启动模式
date: 2019-11-06 14:24:04
tags: Android面试题
---

> 这一系列文章致力于为 Android 开发者查漏补缺，面试准备。
>
> 所有文章首发于公众号「JaqenAndroid」，长期持续更新。
>
> 由于笔者水平有限，总结的答案难免会出现错误，欢迎留言指出，大家一起学习、交流、进步。

众所周知，Activity 有 4 种启动模式，分别是：standard、singleTop、singleTask 和 singleInstance，它们控制了 Activity 的启动行为，不同的启动模式使用于不同的应用场景。

启动的 Activity 会放在任务栈中，任务栈是一种后进先出的结构，按 Back 键的时候栈顶 Activity 会从任务栈中返回，当任务栈为空时系统就会回收这个任务栈。 

本文将通过具体 Demo，详细分析这几种模式的差异和使用场景。

## standard 标准模式

```xml
android:launchMode="standard"
```

Activity 默认的启动模式，每次启动都会创建新的实例，不管这个实例是否已经存在于任务栈。

```
// 启动顺序
MainActivity -> StandardActivity -> StandardActivity

// 栈内容（adb shell dumpsys activity activities）
Running activities (most recent first):
    TaskRecord{b4e290f #18017 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=3}
    Run #2: ActivityRecord{1666db1 u0 com.wuzy.androidlaunchmodetest/.StandardActivity t18017}
    Run #1: ActivityRecord{17fa3e6 u0 com.wuzy.androidlaunchmodetest/.StandardActivity t18017}
    Run #0: ActivityRecord{9e18184 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18017}
```

启动两次 StandardActivity 会创建两个 StandardActivity 的实例对象。

**使用场景**：

在一系列启动 Activity 的过程中需要保留用户操作的 Activity 的页面。比如： 社交应用中，点击查看用户 A 信息 -> 查看用户 A 粉丝 -> 在粉丝中挑选查看用户 B 信息 -> 查看用户 B 粉丝。

## singleTop 栈顶复用模式

```xml
android:launchMode="singleTop"
```

singleTop 与 standard 几乎一样，使用 singleTop 的 Activity 也可以创建多个实例。不同点在于，如果启动的 Activity 已经位于任务栈的栈顶，则不需要创建新的实例，直接复用栈顶的 Activity 实例，intent 通过 Activity 的`onNewIntent` 方法传递到这个 Activity 。

```
例 1：
// 启动顺序
MainActivity -> SingleTopActivity -> SingleTopActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{9de11c2 #18073 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{f0acc0c u0 com.wuzy.androidlaunchmodetest/.SingleTopActivity t18073}
    Run #0: ActivityRecord{b884d57 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18073}
    
-----------------------------------------------------------------------------------------

例 2：
// 启动顺序
MainActivity -> SingleTopActivity -> StandardActivity -> SingleTopActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{9de11c2 #18073 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=4}
    Run #3: ActivityRecord{c282b33 u0 com.wuzy.androidlaunchmodetest/.SingleTopActivity t18073}
    Run #2: ActivityRecord{76fb23e u0 com.wuzy.androidlaunchmodetest/.StandardActivity t18073}
    Run #1: ActivityRecord{b6969a8 u0 com.wuzy.androidlaunchmodetest/.SingleTopActivity t18073}
    Run #0: ActivityRecord{b884d57 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18073}
```

例 1 中由于栈顶已经是 SingleTopActivity，再启动 SingleTopActivity 时直接复用了栈顶 Activity，无需创建新的实例。

例 2 中第二次启动 SingleTopActivity 时，由于栈顶是 StandardActivity，所以启动 SingleTopActivity 时会创建新的实例。

**使用场景**：

假设你在当前的 Activity 中又要启动同类型的 Activity，此时建议将此类型 Activity 的启动模式指定为 singleTop，能够减少 Activity 的创建，节省内存。

## singleTask 栈内复用模式

```xml
android:launchMode="singleTask" 
```

singleTask 标记的 Activity 是栈内复用模式，如果当前任务栈内没有这个 Activity，那么创建新的 Activity，如果当前任务栈内有这个 Activity，不管它在任务栈的哪个位置，都会直接复用这个 Activity，这个 Activity 上面的其他的 Activity 都被移出栈， intent 通过 `onNewIntent` 传递到这个 Activity 。

```
// 1、启动顺序
MainActivity -> SingleTaskActivity -> StandardActivity

// 栈内容：
Running activities (most recent first):
    TaskRecord{ebb2593 #18095 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=3}
    Run #2: ActivityRecord{49b6bb8 u0 com.wuzy.androidlaunchmodetest/.StandardActivity t18095}
    Run #1: ActivityRecord{48628d2 u0 com.wuzy.androidlaunchmodetest/.SingleTaskActivity t18095}
    Run #0: ActivityRecord{86fe71f u0 com.wuzy.androidlaunchmodetest/.MainActivity t18095}

-----------------------------------------------------------------------------------------

// 2、启动顺序
MainActivity -> SingleTaskActivity -> StandardActivity -> SingleTaskActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{ebb2593 #18095 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{48628d2 u0 com.wuzy.androidlaunchmodetest/.SingleTaskActivity t18095}
    Run #0: ActivityRecord{86fe71f u0 com.wuzy.androidlaunchmodetest/.MainActivity t18095}
```

可以看到，在第二次启动 SingleTaskActivity 时，由于栈内已经存在了 SingleTaskActivity  实例，栈顶 StandardActivity 被移出任务栈，复用了栈内 SingleTaskActivity 实例。

当以 singleTask 启动一个 Activity 的时候，首先去判断是否要为该 Activity 去创建一个任务栈？如果需要的话，那么就会创建一个任务栈，并且将该 Activity 放入栈中；如果不需要的话，直接将该 Activity 放入当前的任务栈中。 

那么如何判断要不要为 singleTask Activity 创建一个任务栈？

任务栈的创建跟 taskAffinity 的属性相关，每个 Activity 都有 taskAffinity 属性，这个属性指出了它希望进入的任务栈。如果一个 Activity 没有显式的指明该 Activity 的 taskAffinity，那么它的这个属性就等于 Application 指明的 taskAffinity，如果 Application 也没有指明，那么该 taskAffinity 的值就等于包名。

这里我指定一下 Activity 的 taskAffinity ：

```xml
<activity
    android:name=".SingleTaskWithAffinityActivity"
    android:label="SingleTaskWithAffinity Activity"
    android:launchMode="singleTask"
    android:taskAffinity="com.jaqen" />
```

看一下测试结果：

```
// 1、启动顺序
MainActivity -> SingleTaskWithAffinityActivity

// 任务栈
Running activities (most recent first):
    TaskRecord{fa7e695 #18097 A=com.jaqen U=0 StackId=1 sz=1}
    Run #1: ActivityRecord{6267f33 u0 com.wuzy.androidlaunchmodetest/.SingleTaskWithAffinityActivity t18097}
    TaskRecord{efcc3aa #18096 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{ccdada8 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18096}
    
-----------------------------------------------------------------------------------------

// 2、启动顺序
MainActivity -> SingleTaskWithAffinityActivity -> StandardActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{3f4af35 #18097 A=com.jaqen U=0 StackId=1 sz=2}
    Run #2: ActivityRecord{23b4c1a u0 com.wuzy.androidlaunchmodetest/.StandardActivity t18097}
    Run #1: ActivityRecord{d234ee0 u0 com.wuzy.androidlaunchmodetest/.SingleTaskWithAffinityActivity t18097}
    TaskRecord{e27d53b #18096 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{f445d23 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18096}

-----------------------------------------------------------------------------------------

// 3、启动顺序
MainActivity -> SingleTaskWithAffinityActivity -> StandardActivity -> SingleTaskWithAffinityActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{3f4af35 #18097 A=com.jaqen U=0 StackId=1 sz=1}
    Run #1: ActivityRecord{d234ee0 u0 com.wuzy.androidlaunchmodetest/.SingleTaskWithAffinityActivity t18097}
    TaskRecord{e27d53b #18096 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{f445d23 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18096}
```

首次启动 SingleTaskWithAffinityActivity 会创建新的任务栈（*大括号内 `#` 后的数字标识任务栈 id*）。

在 SingleTaskWithAffinityActivity 启动 StandardActivity ， 这个 StandardActivity 与 SingleTaskWithAffinityActivity 在同一个栈。

SingleTaskWithAffinityActivity  会出现在多任务界面。

第二次启动 SingleTopActivity 时直接复用了栈内已存 Activity，已存 Activity 上的 Activity 被移出任务栈。

**使用场景**：

一般应用主页面可以用 singleTask 方式。比如用户在主页跳转到其他页面，运行多次操作后想返回到主页。

## singleInstance 单实例模式

```xml
android:launchMode="singleInstance"
```

singleInstance 与 singleTask 类似，在应用都只存在一个实例，不同点在于存放 singleInstance Activity 实例的任务栈只能存放唯一的 singleInstance Activity。

```
// 1、启动顺序
MainActivity -> SingleInstanceActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{cd22626 #18116 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #1: ActivityRecord{462d9a2 u0 com.wuzy.androidlaunchmodetest/.SingleInstanceActivity t18116}
    TaskRecord{c2b08bd #18115 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{812c79c u0 com.wuzy.androidlaunchmodetest/.MainActivity t18115}
    
-----------------------------------------------------------------------------------------

// 2、启动顺序
MainActivity -> SingleInstanceActivity -> StandardActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{e46fd18 #18115 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #2: ActivityRecord{540b885 u0 com.wuzy.androidlaunchmodetest/.StandardActivity t18115}
    TaskRecord{5cab9d7 #18116 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #1: ActivityRecord{18780a3 u0 com.wuzy.androidlaunchmodetest/.SingleInstanceActivity t18116}
    TaskRecord{e46fd18 #18115 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #0: ActivityRecord{6ceacf3 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18115}
    
-----------------------------------------------------------------------------------------

// 3、启动顺序
MainActivity -> SingleInstanceActivity -> StandardActivity -> SingleInstanceActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{e46fd18 #18115 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #2: ActivityRecord{540b885 u0 com.wuzy.androidlaunchmodetest/.StandardActivity t18115}
    TaskRecord{5cab9d7 #18116 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #1: ActivityRecord{18780a3 u0 com.wuzy.androidlaunchmodetest/.SingleInstanceActivity t18116}
    TaskRecord{e46fd18 #18115 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #0: ActivityRecord{6ceacf3 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18115}
```

启动 SingleInstanceActivity 会创建新的任务栈，从 SingleInstanceActivity 中启动 StandardActivity，StandardActivity 会被放到上一个任务栈中。

再此启动 SingleInstanceActivity，SingleInstanceActivity 会被复用。

**使用场景**：

singleInstance 模式常应用于独立栈操作的应用，如闹钟的提醒页面，当你在A应用中看视频时，闹钟响了，你点击闹钟提醒通知后进入提醒详情页面，然后点击返回就再次回到A的视频页面，这样就不会过多干扰到用户先前的操作了。 

## Intent Flags

 除了在 manifest 文件中设置 launchMode 之外，还可以在 Intent 中设置 Flag 达到同样的效果。

常见几种 Flag：

1、**FLAG_ACTIVITY_NEW_TASK**

 在 google 的官方文档中介绍，它与 `launchMode="singleTask"` 具有相同的行为。实际上，并不是完全相同！具体看下面的案例分析。 

2、**FLAG_ACTIVITY_SINGLE_TOP**

 等同于 `launchMode="singleTop"` 。

3、**FLAG_ACTIVITY_CLEAR_TOP**

 清除包含目标 Activity 的任务栈中位于该 Activity 实例之上的其他 Activity 实例。 但是是复用已有的目标 Activity，还是先删除后重建，则有以下规则： 

- 若是使用 FLAG_ACTIVITY_SINGLE_TOP 和 FLAG_ACTIVITY_CLEAR_TOP 标志位组合，那么不管目标 Activity 是什么启动模式，都会被复用。 

- 若是单独使用 FLAG_ACTIVITY_CLEAR_TOP，那么只有非 standard 启动模式的目标 Activity 才会被复用，否则都先被删除，然后被重新创建并入栈。 

4、**FLAG_ACTIVITY_CLEAR_TASK** 

首先清空已经存在的目标 Activity 实例所在的任务栈，这自然也就清除了之前存在的目标 Activity 实例，然后创建新的目标 Activity 实例并入栈。

通过几个案例查看 Flag 的使用效果。

- MainActivity 为 standard 模式，未设置 Flag。

- IntentFlagTestActivity 为 standard 模式，未设置 taskAffinity。

```xml
<activity
   android:name=".IntentFlagTestActivity"
   android:label="IntentFlagTestActivity" />
```

- IntentFlagTestWithAffinityActivity 为 standard 模式，设置与 MainActivity 不同的 taskAffinity。 

```xml
<activity android:name=".IntentFlagTestWithAffinityActivity"
    android:taskAffinity="com.jaqen"
    android:label="IntentFlagTestWithAffinityActivity"/> 
```

### 1、单独使用 FLAG_ACTIVITY_NEW_TASK

- taskAffinity 相同时：

```java
// 启动 IntentFlagTestActivity 的方式
Intent intent = new Intent(MainActivity.this, IntentFlagTestActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

测试结果：

```
// 启动流程
MainActivity -> IntentFlagTestActivity -> MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
TaskRecord{89317d5 #18128 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=4}
Run #3: ActivityRecord{2f1ac92 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18128}
Run #2: ActivityRecord{a0104eb u0 com.wuzy.androidlaunchmodetest/.MainActivity t18128}
Run #1: ActivityRecord{9b84b56 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18128}
Run #0: ActivityRecord{f9a57f4 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18128}
```

从任务栈可以看出，在  taskAffinity 相同的情况下，单独使用 FLAG_ACTIVITY_NEW_TASK 不会产生任何效果！

- taskAffinity 不同时：

```java
// 启动 IntentFlagTestWithAffinityActivity 的方式
Intent intent = new Intent(MainActivity.this, IntentFlagTestWithAffinityActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

测试结果：

```
// 启动流程
MainActivity -> IntentFlagTestWithAffinityActivity -> MainActivity -> IntentFlagTestWithAffinityActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{d07e70 #18135 A=com.jaqen U=0 StackId=1 sz=2}
    Run #2: ActivityRecord{997119c u0 com.wuzy.androidlaunchmodetest/.MainActivity t18135}
    Run #1: ActivityRecord{8a7f641 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestWithAffinityActivity t18135}
    TaskRecord{84f526e #18134 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{5aca49d u0 com.wuzy.androidlaunchmodetest/.MainActivity t18134}
```

在 taskAffinity 不同的情况下， 添加 FLAG_ACTIVITY_NEW_TASK 确实产生了一些效果，第一次启动 IntentFlagTestWithAffinityActivity 创建了新的任务栈，但是第二次从 MainActivity 中启动 IntentFlagTestWithAffinityActivity  时，没有任何反应。

**结论：**

**单独使用 FLAG_ACTIVITY_NEW_TASK 并不会产生与 singleTask 相同的效果**。

### 2、FLAG_ACTIVITY_NEW_TASK + FLAG_ACTIVITY_CLEAR_TOP

- taskAffinity 相同时：

```java
// 启动 IntentFlagTestActivity 的方式
Intent intent = new Intent(MainActivity.this, IntentFlagTestActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TOP);
startActivity(intent);
```

测试结果：

```
// 1、启动流程
MainActivity -> IntentFlagTestActivity -> MainActivity
    
// 栈内容
Running activities (most recent first):
    TaskRecord{7bb1982 #18139 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=3}
    Run #2: ActivityRecord{535c0d0 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18139}
    Run #1: ActivityRecord{c5253ff u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18139}
    Run #0: ActivityRecord{1b04db1 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18139}

-----------------------------------------------------------------------------------------

// 2、启动流程
MainActivity -> IntentFlagTestActivity -> MainActivity -> IntentFlagTestActivity
    
// 栈内容
Running activities (most recent first):
    TaskRecord{7bb1982 #18139 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{705bf3 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18139}
    Run #0: ActivityRecord{1b04db1 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18139}
```

在 taskAffinity 相同情况下，FLAG_ACTIVITY_NEW_TASK  + FLAG_ACTIVITY_CLEAR_TOP 不会创建新的任务栈。

貌似和 singleTask 启动模式效果相同，但是细看会发现区别：前后两次 IntentFlagTestActivity 并不是同一个实例，也就是并没有复用栈内的 IntentFlagTestActivity，而是清除了 IntentFlagTestActivity 本身及其之上的所有 Activity，然后新建 IntentFlagTestActivity 实例添加到当前任务栈。

- taskAffinity 不同时：

```java
// 启动 IntentFlagTestWithAffinityActivity 的方式
Intent intent = new Intent(MainActivity.this, IntentFlagTestWithAffinityActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TOP);
startActivity(intent);
```

测试结果：

```
// 1、启动流程
MainActivity -> IntentFlagTestWithAffinityActivity -> MainActivity

// 任务栈
Running activities (most recent first):
    TaskRecord{a1b1a38 #18152 A=com.jaqen U=0 StackId=1 sz=2}
    Run #2: ActivityRecord{ae8c352 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18152}
    Run #1: ActivityRecord{5647f4a u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestWithAffinityActivity t18152}
    TaskRecord{e2f8776 #18151 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{864a2f5 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18151}

-----------------------------------------------------------------------------------------

// 2、启动流程
MainActivity -> IntentFlagTestWithAffinityActivity -> MainActivity -> IntentFlagTestWithAffinityActivity
    
Running activities (most recent first):
    TaskRecord{a1b1a38 #18152 A=com.jaqen U=0 StackId=1 sz=1}
    Run #1: ActivityRecord{cf5fce6 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestWithAffinityActivity t18152}
    TaskRecord{e2f8776 #18151 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{864a2f5 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18151}
```

可见，与 taskAffinity 相同类似（除了创建新的任务栈），在第二次启动 IntentFlagTestWithAffinityActivity 时也是直接清除了 IntentFlagTestWithAffinityActivity 自身及其之上所有的 Activity，然后创建新的 IntentFlagTestWithAffinityActivity 实例添加到任务栈中。

**结论：**

**`FLAG_ACTIVITY_NEW_TASK + FLAG_ACTIVITY_CLEAR_TOP` 标志位组合产生的效果总体上和 singleTask 模式相同，但不会复用 Activity。**

### 3、FLAG_ACTIVITY_NEW_TASK + FLAG_ACTIVITY_CLEAR_TASK

- taskAffnity 相同时：

```java
// 启动 IntentFlagTestActivity 的方式
Intent intent = new Intent(MainActivity.this, IntentFlagTestActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
startActivity(intent);
```

测试结果:

```
// 1、启动流程
MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{572ae8d #18253 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{e0aa3a u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18253}

-----------------------------------------------------------------------------------------

// 2、启动流程
MainActivity -> IntentFlagTestActivity  -> MainActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{572ae8d #18253 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{65fb5c2 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18253}
    Run #0: ActivityRecord{e0aa3a u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18253}

-----------------------------------------------------------------------------------------

// 3、启动流程
MainActivity -> IntentFlagTestActivity  -> MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{572ae8d #18253 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{9eccfa3 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18253}
	
```

可见， 当通过`FLAG_ACTIVITY_NEW_TASK + FLAG_ACTIVITY_CLEAR_TASK ` 标志位组合启动 IntentFlagTestActivity 时，首先会清空 IntentFlagTestActivity 所在的任务栈，然后再创建新的 IntentFlagTestActivity 实例并入栈。 

- taskAffnity 不同时：

```java
// 启动 IntentFlagTestWithAffinityActivity 的方式
Intent intent = new Intent(MainActivity.this, IntentFlagTestWithAffinityActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
startActivity(intent);
```

测试结果:

```
// 1、启动流程
MainActivity -> IntentFlagTestWithAffinityActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{c081b17 #18257 A=com.jaqen U=0 StackId=1 sz=1}
    Run #1: ActivityRecord{8f8a3c5 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestWithAffinityActivity t18257}
    TaskRecord{60908ed #18256 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{5924b86 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18256}
    
-----------------------------------------------------------------------------------------

// 2、启动流程
MainActivity -> IntentFlagTestWithAffinityActivity -> MainActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{c081b17 #18257 A=com.jaqen U=0 StackId=1 sz=2}
    Run #2: ActivityRecord{7e1a8a0 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18257}
    Run #1: ActivityRecord{8f8a3c5 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestWithAffinityActivity t18257}
    TaskRecord{60908ed #18256 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{5924b86 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18256}

-----------------------------------------------------------------------------------------

// 3、启动流程
MainActivity -> IntentFlagTestWithAffinityActivity -> MainActivity -> IntentFlagTestWithAffinityActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{c081b17 #18257 A=com.jaqen U=0 StackId=1 sz=1}
    Run #1: ActivityRecord{e52a3c0 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestWithAffinityActivity t18257}
    TaskRecord{60908ed #18256 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=1}
    Run #0: ActivityRecord{5924b86 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18256}
```

结果与 taskAffnity 相同情况下类似， 首先会清空 IntentFlagTestWithAffinityActivity 所在的任务栈，然后再创建新的 IntentFlagTestWithAffinityActivity 实例并入栈，这和 taskAffinity 属性相同是一致的效果，只不过这里第一次为 IntentFlagTestWithAffinityActivity 创建了新的任务栈。 

**结论：**

**`FLAG_ACTIVITY_NEW_TASK + FLAG_ACTIVITY_CLEAR_TASK ` 标志位组会先清空任务栈，再创建新的 Activity 实例入栈。**

### 4、单独使用 FLAG_ACTIVITY_CLEAR_TOP

- IntentFlagTestActivity 启动模式：standard

```
// 1、启动流程
MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{b6045f3 #18282 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{44dcf5f u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18282}
    Run #0: ActivityRecord{c713f21 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18282}

-----------------------------------------------------------------------------------------

// 2、启动流程
MainActivity -> IntentFlagTestActivity  -> MainActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{b6045f3 #18282 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=3}
    Run #2: ActivityRecord{13c806b u0 com.wuzy.androidlaunchmodetest/.MainActivity t18282}
    Run #1: ActivityRecord{44dcf5f u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18282}
    Run #0: ActivityRecord{c713f21 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18282}

-----------------------------------------------------------------------------------------

// 3、启动流程
MainActivity -> IntentFlagTestActivity  -> MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{b6045f3 #18282 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{fa6320c u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18282}
    Run #0: ActivityRecord{c713f21 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18282}
```

- IntentFlagTestActivity 启动模式：singleTask

```
// 1、启动流程
MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{7c9d493 #18280 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{daafb1e u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18280}
    Run #0: ActivityRecord{b547fca u0 com.wuzy.androidlaunchmodetest/.MainActivity t18280}

-----------------------------------------------------------------------------------------

// 2、启动流程
MainActivity -> IntentFlagTestActivity  -> MainActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{7c9d493 #18280 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=3}
    Run #2: ActivityRecord{511762c u0 com.wuzy.androidlaunchmodetest/.MainActivity t18280}
    Run #1: ActivityRecord{daafb1e u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18280}
    Run #0: ActivityRecord{b547fca u0 com.wuzy.androidlaunchmodetest/.MainActivity t18280}

-----------------------------------------------------------------------------------------

// 3、启动流程
MainActivity -> IntentFlagTestActivity  -> MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{7c9d493 #18280 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{daafb1e u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18280}
    Run #0: ActivityRecord{b547fca u0 com.wuzy.androidlaunchmodetest/.MainActivity t18280}
```

从上面两个例子看出，单独使用 FLAG_ACTIVITY_CLEAR_TOP 时，

standard 启动模式下，目标 Activity 自身及其上的 Activity 都会被销毁，目标 Activity 自身会重新创建放入栈中；singleTask 启动模式下，先销毁目标 Activity 之上的所有 Activity，然后复用已有的 Activity。

此外，singleTop、singleInstance 与 singleTask 一样，都会复用已有 Activity。这里不在赘述。

**结论：**

**单独使用 FLAG_ACTIVITY_CLEAR_TOP，那么只有非 standard 启动模式的目标 Activity 才会被复用，否则都先被删除，然后被重新创建并入栈。**

### 5、FLAG_ACTIVITY_CLEAR_TOP  + FLAG_ACTIVITY_SINGLE_TOP

- IntentFlagTestActivity 启动模式 standard

```
// 1、启动流程
MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{bf65b55 #18326 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{c62eb86 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18326}
    Run #0: ActivityRecord{eb38b03 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18326}


-----------------------------------------------------------------------------------------

// 2、启动流程
MainActivity -> IntentFlagTestActivity  -> MainActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{bf65b55 #18326 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=3}
    Run #2: ActivityRecord{be92d5b u0 com.wuzy.androidlaunchmodetest/.MainActivity t18326}
    Run #1: ActivityRecord{c62eb86 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18326}
    Run #0: ActivityRecord{eb38b03 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18326}

-----------------------------------------------------------------------------------------

// 3、启动流程
MainActivity -> IntentFlagTestActivity  -> MainActivity -> IntentFlagTestActivity

// 栈内容
Running activities (most recent first):
    TaskRecord{bf65b55 #18326 A=com.wuzy.androidlaunchmodetest U=0 StackId=1 sz=2}
    Run #1: ActivityRecord{c62eb86 u0 com.wuzy.androidlaunchmodetest/.IntentFlagTestActivity t18326}
    Run #0: ActivityRecord{eb38b03 u0 com.wuzy.androidlaunchmodetest/.MainActivity t18326}
```

`FLAG_ACTIVITY_CLEAR_TOP  + FLAG_ACTIVITY_SINGLE_TOP` 标志位组合情况， standard 模式下的 IntentFlagTestActivity 被复用了， 那么其他启动模式的 Activity 也必然会被复用。（单独使用 FLAG_ACTIVITY_CLEAR_TOP 都会被复用，何况又添加了 FLAG_ACTIVITY_SINGLE_TOP 标志位，通过 Demo 验证也确实如此，就不再给出具体案例了）。 

**结论：**

**使用 FLAG_ACTIVITY_SINGLE_TOP 和 FLAG_ACTIVITY_CLEAR_TOP 标志位组合，那么不管目标 Activity 是什么启动模式，都会被复用。** 

OK，Activity 启动模式相关的内容就介绍这些，希望感兴趣的朋友有帮助。

Demo 我已经放在了 GitHub 上，有兴趣可以下载下来，运行看看结果。

 https://github.com/zywudev/AndroidLaunchModeTest