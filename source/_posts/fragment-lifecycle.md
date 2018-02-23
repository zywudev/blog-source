---
title: Android 碎片生命周期
date: 2017-03-27 15:55:34
---
本文目的在于总结 Android 碎片的生命周期。

## 碎片的生命周期

### 碎片的运行状态

1、**运行状态**： 碎片可见，并且与其相关联的活动是运行状态。
2、**暂停状态**： 活动进入暂停状态，关联的碎片也进入暂停状态。
3、**停止状态**： 当一个活动进入停止状态，相关联的碎片也进入停止状态；通过调用  FragmentTransaction 的 remove() 或者 replace() 方法移除或替换，在事务提交之前将碎片添加到返回栈会使碎片进入停止状态。
4、**销毁状态**： 当一个活动被销毁，相关联的碎片被销毁；通过调用  FragmentTransaction 的 remove() 或者 replace() 方法移除或替换，在事务提交之前未将碎片添加到返回栈会使碎片进入销毁状态。

### 回调方法

活动中的回调方法碎片中都有，此外碎片中还附加了一些回调方法。
1、**onAttach()**： 当碎片与活动建立关联时调用。
2、**onCreateView()**： 为碎片创建视图时调用。
3、**onActivityCreated()**： 确保与碎片相关联的活动一定已经创建完毕的时候调用。
4、**onDestroyView()**： 当与碎片关联的视图被移除的时候调用。
5、**onDetach()**： 当碎片与活动解除关联时调用。

Android [官网](https://developer.android.google.cn/guide/components/fragments.html?hl=zh-cn#Lifecycle) 提供了碎片生命周期图。
![](http://om9o63aks.bkt.clouddn.com/fragment_lifecycle.png)

## Demo 实例
通过 demo 演示碎片的生命周期。

1、新建 FragmentTest 项目。
2、创建三个碎片，LeftFragment、RightFragment 和 AnotherFragment，如下图，当点击左边碎片中的按钮，右边布局从 RightFragment 切换到 AnotherFragment。
![](http://om9o63aks.bkt.clouddn.com/fragment_demo.png)
3、重写碎片的回调方法，演示 RightFragment 的生命周期。

```java
package com.vincent.fragmenttest;

import android.content.Context;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

/**
 * Created by Wu on 2017-03-27.
 */

public class RightFragment extends Fragment {

    public static final String TAG = "RightFragment";

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        Log.e(TAG, "onAttach");
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.e(TAG, "onCreate");
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        Log.e(TAG, "onCreateView");
        View view = inflater.inflate(R.layout.right_fragment, container, false);
        return view;
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        Log.e(TAG, "onActivityCreated");
    }

    @Override
    public void onStart() {
        super.onStart();
        Log.e(TAG, "onStart");
    }

    @Override
    public void onResume() {
        super.onResume();
        Log.e(TAG, "onResume");
    }

    @Override
    public void onPause() {
        super.onPause();
        Log.e(TAG, "onPause");
    }

    @Override
    public void onStop() {
        super.onStop();
        Log.e(TAG, "onStop");
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        Log.e(TAG, "onDestroyView");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e(TAG, "onDestroy");
    }

    @Override
    public void onDetach() {
        super.onDetach();
        Log.e(TAG, "onDetach");
    }
}

```
当 RightFragment 第一次加载到屏幕上，依次执行 `onAttach() - onCreate() - onCreateView() - onActivityCreated() - onStart() - onResume()` 方法。
![](http://om9o63aks.bkt.clouddn.com/fragment_start.png)

然后点击 LeftFragment 中的按钮， AnotherFragment 载入屏幕，依次执行 `onPause() - onStop() - onDestroyView()` 方法，此时 RightFragment 进入停止状态，如果在替换时未将 RightFragment 添加到返回栈，则 RightFragment 会进入销毁状态。
![](http://om9o63aks.bkt.clouddn.com/fragment_button.png)

按下返回键，RightFragment 重写载入屏幕,依次执行 `onCreateView() - onActivityCreated() - onStart() - onResume()` 方法。
![](http://om9o63aks.bkt.clouddn.com/fragment_restart.png)

再次按下返回键，依次执行 `onPause() - onStop() - onDestroyView() - onDestroy() - onDetach()` 方法。

[源码下载](https://github.com/zywudev/blog-source/tree/master/FragmentTest)

## 参考资料
[Android官方文档](https://developer.android.google.cn/guide/components/fragments.html?hl=zh-cn#Lifecycle)
[<第一行代码> 郭霖](https://www.amazon.cn/dp/B01MSR5D04)

（完）