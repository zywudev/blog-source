---
title: Android onSaveInstanceState
date: 2018-06-09 21:20:09
tags:
---

1、onSaveInstanceState 调用时机？

- 按 home 键退到后台
- 按手机息屏键
- 选择其他程序时
- 从当前 Activity 启动其他 Activity 中
- 屏幕切换时
- 系统内存不足时，优先级低的 Activity 被杀死时

总之，onSaveInstanceState 的调用遵循一个重要原则，即当系统 ”未经许可“ 有销毁 Activity 的可能时，系统有责任保存一些非永久性的数据，而正常退出时是不会调用的。

2、onSaveInstanceState 怎样保存数据？

- 系统调用 onSaveInstanceState 保存 Activity 的视图结构，比如文本框的输入数据。系统的工作流程大致是，Activity 委托 Window 去保存数据，Window 再委托 DecorView 保存数据， DecorView 再委托各个子元素保存数据。
- 当然如果你想保存一些想要的数据，需要重写 onSaveInstanceState 方法。

3、onRestoreInstanceState 调用时机？

- onSaveInstanceState 方法和 onRestoreInstanceState 方法 “不一定” 是成对的被调用的。
- onRestoreInstanceState 被调用的前提是，Activity 确实被系统销毁了，而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用，例如，当正在显示 Activity 的时候，用户按下 home 键回到主界面，然后用户紧接着又返回到 Activity，这种情况下 Activity 一般不会因为内存的原因被系统销毁，故Activity 的onRestoreInstanceState 方法不会被执行。
- 而比如屏幕切换时，Activity 会先被销毁然后重建，saveInstanceState 方法里保存的数据都会传到 onCreate 和 onRestoreInstanceState 方法中，如果在 onCreate 中恢复数据，需要先判断 Bundle 是否为空。而在 onRestoreInstanceState  方法中不需要判空处理，因为这个方法只会在有数据需要恢复时才被调用，Bundle 不可能为空。所以推荐在这个方法中恢复数据。