---
title: Android 屏幕适配笔记
date: 2017-08-02 13:19:11
tag: Android 基础
categories: Android
---

## 基本概念

- in

英寸，手机屏幕的物理尺寸。1 英寸等于 2.54cm。如果说手机屏幕是 5 寸，表示手机屏幕对角线长度为 5X2.54=12.7cm。

- px

像素，英文单词 pixel 的缩写，屏幕上的点。常见的分辨率 320x480、480x800、720x1280、1080x1920 指的就是像素。

- dpi

每英寸包含的像素个数，dots per inch 的缩写。比如 320X480 分辨率的手机，宽 2 英寸，高 3 英寸, 每英寸包含的像素点的数量为 320/2=160dpi（横向）或480/3=160dpi（纵向），160就是这部手机的dpi。

- density

屏幕密度，density 和 dpi 的关系为 density = dpi/160。

- dp

独立密度像素，density independent pixel 的缩写。在 Android 中，规定以 160dpi 为基准，1dip = 1px，如果密度是 320dpi，则1dp = 2px，以此类推。

- sp

独立比例像素，scale independent pixel 的缩写。Android 中设置字体大小，不推荐奇数，容易造成精度丢失。

## 解决方案

### 使用 wrap_content、match_parent

要确保布局的灵活性并适应各种尺寸的屏幕，应该使用 “wrap_content” 和 “match_parent” 控制某些视图组件的宽度和高度。使用 “wrap_content”，系统就会将视图的宽度或高度设置成刚好能够包含视图中的内容，而 “match_parent”（在低于 API 级别 8 的级别中称为 “fill_parent”）则会让视图的宽和高延伸至充满整个父布局。

### 使用 RelativeLayout

RelativeLayout 允许布局的子控件之间使用相对定位的方式控制控件的位置，比如可以让一个子视图居屏幕左侧对齐，让另一个子视图居屏幕右侧对齐。

### 使用 Size 限定符

配置 Size 限定符允许程序在运行时根据当前设备的配置自动加载合适的资源（比如为不同尺寸屏幕设计不同的布局）。

`res/layout/main.xml`，单面板（默认）布局：

```html
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="match_parent" />
</LinearLayout>
```

`res/layout-large/main.xml`，双面板布局：

```html
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="400dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
```

第二个布局的目录名中包含了 large 限定符，那些被定义为大屏的设备（比如 7 寸以上的平板）会自动加载此布局，而小屏设备会加载另一个默认的布局。

### 使用 Smallest-width 限定符

Smallest-width 限定符允许设定一个具体的最小值(以 dp 为单位)来指定屏幕。例如，7 寸的平板最小宽度是 600dp。

`res/layout/main.xml`，single-pane（默认）布局：

```html
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="match_parent" />
</LinearLayout>
```

`res/layout-sw600dp/main.xml`，双面板布局：

```html
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal">
    <fragment android:id="@+id/headlines"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.HeadlinesFragment"
              android:layout_width="400dp"
              android:layout_marginRight="10dp"/>
    <fragment android:id="@+id/article"
              android:layout_height="fill_parent"
              android:name="com.example.android.newsreader.ArticleFragment"
              android:layout_width="fill_parent" />
</LinearLayout>
```

那些最小屏幕宽度大于 600dp 的设备会选择 layout-sw600dp/main.xml(two-pane) 布局，而更小屏幕的设备将会选择 layout/main.xml(single-pane) 布局。Android 3.2 系统的设备无法识别 sw600dp 这个限定符，所以同时需要使用 large 限定符。

### 使用布局别名

为了避免 Smallest-width 限定符为兼容 Android 3.2 之前系统而重复定义布局，可以使用布局别名技巧。

首先定义

- res/layout/main.xml，single-pane 布局
- res/layout/main_twopanes.xml，two-pane 布局

加入以下两个文件：

`res/values-large/layout.xml`：

```html
<resources>  
	<item name="main" type="layout">@layout/main_twopanes</item>  
</resources> 
```

`res/values-sw600dp/layout.xml`：

```html
<resources>
    <item name="main" type="layout">@layout/main_twopanes</item>
</resources>
```

### 使用 Orientation 限定符

根据屏幕方向进行布局的调整。

### 使用 Nine-Patch 图片

支持不同屏幕大小通常情况下也意味着图片资源也需要有自适应的能力。例如，一个按钮的背景图片必须能够随着按钮大小的改变而改变。
使用 nine-patch 图片，它是一种被特殊处理过的 PNG 图片，可以指定哪些区域可以拉伸而哪些区域不可以。

## 参考

- [http://blog.csdn.net/guolin_blog/article/details/8830286](http://blog.csdn.net/guolin_blog/article/details/8830286)
- http://blog.csdn.net/carson_ho/article/details/51234308

