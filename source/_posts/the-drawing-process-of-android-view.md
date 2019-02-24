---
title: Android View 的绘制过程
date: 2018-10-16 16:54:55
tags: Android
toc: true
---

## View 整体结构

Activity、Window、DecorView 之间的关系：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/FsefzvUt3y5K6D1f262mDFGNwRsp.png)

**Activity**: 类似控制器，统筹视图的添加与显示，以及通过回调来与 Window、View 进行交互。

**Window**：视图承载器，抽象类，PhoneWindow 是其唯一实现类。

**DecorView**：顶级 View，包含 StatusBar、TitleBar + ContentView 和 NavigationBar。

StatusBar 是状态栏； 

TitleBar 对应各种 ActionBar； 

ContentView 对应 R.id.content，setContentView 设置的 View 被添加到 R.id.content 对应的 View 上。

NavigationBar是虚拟按键。

**ViewRoot**: 实现类是 ViewRootImpl，它是连接 WidowManager 和 DecorView 的纽带。View 的三大流程(测量（measure），布局（layout），绘制（draw）)均通过 ViewRoot 来完成。ViewRoot 继承了 Handler 类，可以接收事件并分发，Android 的所有触屏事件、按键事件、界面刷新等事件都是通过 ViewRoot 进行分发的。

## View 的工作流程

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/FuMzXGfmOHg5VKedzv8MvIlE9JpA.png)

View 的绘制是从上往下一层层迭代，DecorView –> ViewGroup（—>ViewGroup）–> View ，依次 measure、layout 、draw。

### measure 

**ViewGroup.LayoutParams**

布局参数类，指定视图 View 的高度和宽度等参数。

**MeasureSpec**

测量规格类，决定一个视图的大小。

**measureSpec**：32 位 int 值，高 2 位代表 mode，低 30 位代表 size。

**mode**：测量模式，包括：

- UNSPECIFIED：父 View 不约束当前 View 的大小。
- EXACTLY：对应 LayoutParams 中的 match_parent 和具体数值这两种模式。父 View 决定当前 View 的精确大小。
- AT_MOST：对应 LayoutParams 中的 wrap_content，View 的大小不能大于父容器的大小。

**size**：某种测量模式下的规格大小。

```java
public static class MeasureSpec {
    private static final int MODE_SHIFT = 30;
    private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

    public static final int UNSPECIFIED = 0 << MODE_SHIFT;
    public static final int EXACTLY     = 1 << MODE_SHIFT;
    public static final int AT_MOST     = 2 << MODE_SHIFT;

    public static int makeMeasureSpec(int size, int mode) {
        return (size & ~MODE_MASK) | (mode & MODE_MASK);
    }

    public static int getMode(int measureSpec) {
        return (measureSpec & MODE_MASK);
    }

    public static int getSize(int measureSpec) {
        return (measureSpec & ~MODE_MASK);
    }
}
```

**measure 流程**

 1、ViewRootImpl.performMeasure -> performMeasure()

根据手机屏幕的宽高和 DecorView 的 LayoutParams 生成 DecorView 的 MeasureSpec，然后调用 DecorView 的 measure() 开始 DecorView 的测量。

2、DecorView.measure() -> onMeasure()

DecorView 继承自 FrameLayout，所以会走到 FrameLayout 的 onMeasure() 方法，onMeasure() 里面会调用 measureChild() 为 ViewGroup 生成 MeasureSpec，通过 ViewGroup 的 measure()  开始 ViewGroup 的测量。

3、ViewGroup.measure() -> onMeasure()

如果自定义的 ViewGroup 没有重写 onMeasure() 方法，默认会调用 View.onMeasure() 方法，则不会继续对子 view 进行测量。

因此，自定义 ViewGroup，需要重写 onMeasure() 方法，里面调用 measureChild() 为子 View 生成 MeasureSpec 并测量 child。最后调用 setMeasuredDimension 来设置自己的宽高。

4、View.measure() -> onMeasure()

根据父 View 的 MeasureSpec 和自身的 LayoutParams 参数进行测量。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/Fo1gJQWDZccIp1nk-b72HlBkAfLd.png)

**细节：**

**父 View 的测量方法**：

根据子 View 的布局样式，调用 setMeasuredDimension 来设置自己的宽高。

**子 View 的测量方法**：

根据父 View 的 MeasureSpec 和 自身的 LayoutParams 参数进行测量。

先计算子 View 的MeasureSpec，即 childMeasureSpec；

再调用 child.measure(childWidthMeasureSpec, childHeightMeasureSpec) 测量子 View 的大小。

```java
protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams();

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                                                          mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                                                           mPaddingTop + mPaddingBottom, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

childMeasureSpec 的计算：

由 parentMeasureSpec 和 childDimension 确定。childDimension 为 LayoutParams 的 width 和 height。

**规律**：

- 当子 View 采用具体数值
  - mode: EXACTLY
  - size: 自身设置的具体数值
- 当子 View 采用 match_parent
  - mode: 父容器的测量模式
  - size: 若父容器为EXACTLY，则大小为父容器的剩余大小；若父容器为 AT_MOST，则大小为不超过父容器的剩余大小。
- 当子 View采用 wrap_content
  - mode：AT_MOST
  - size：不超过父容器的剩余空间

### layout

计算视图的位置，也就是 left、top、right、bottom。这些坐标都是相对于父布局的坐标。

布局也是自上而下，不同的是 ViewGroup 先在 layout() 中确定自己的布局，然后在 onLayout() 方法中再调用子View 的 layout() 方法，让子 View 布局。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/FuyoJtSyZy8NbilQKC6f4aU_VdPz.png)

### draw

draw 主要流程：

- 绘制背景 background.draw(canvas)
- 绘制自己（onDraw）
- 绘制Children(dispatchDraw)
- 绘制装饰（onDrawScrollBars）

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/FuMME_uu0Sb6lbxjXmPherQOYs_k.png)

## 参考文档

- [Window、Activity、DecorView以及ViewRoot之间的关系](https://love2.io/@funkkiid/doc/android_interview//android/basis/decorview.md)  

- [View测量、布局及绘制原理](https://love2.io/@funkkiid/doc/android_interview//android/basis/custom_view.md)



















