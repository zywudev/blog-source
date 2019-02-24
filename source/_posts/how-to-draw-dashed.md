---
title: Android 绘制虚线
date: 2018-04-17 19:24:41
tags: Android
toc: true
---

Android 中绘制虚线可采用添加在布局文件中添加一个 View，然后给这个 View 加个虚线背景即可。

一般都是用 shape 绘制虚线 :

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="line">

    <stroke
        android:dashGap="2dp"
        android:dashWidth="2dp"
        android:color="@color/colorAccent"
        android:width="1dp"/>

</shape>
```

然后在需要绘制虚线的地方加个 View :

```java
<View
    android:layout_width="wrap_content"
    android:layout_height="2dp"
    android:background="@drawable/shape_dashed"/>
```

很简单。

但运行到真机上发现并不是虚线，而是一条实线。网上一堆博客文章都是这种做法，不知道他们都验证过没有。。。

查了一些资料，才知道绘制的虚线变成实线的原因是 dashGap 不支持硬件加速，而我们的手机默认是开启了硬件加速的。因此只需要修改下 View 的属性即可。

```java
<View
    android:layout_width="wrap_content"
    android:layout_height="2dp"
    android:background="@drawable/shape_dashed"
    android:layerType="software"/>
```

此外要注意 View 的高度要比 shape 中定义的高度要大，不然不显示。都是一些细节，虽然没什么技术含量，但最近发现很多 Bug 都是低级错误导致的，都是平时不注重细节的原因。在此记录一下，提醒自己。

 