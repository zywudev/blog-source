---
title: Android APK 打包过程
date: 2018-07-07 20:51:23
tags:
---

在日常开发中，每天都会点击 Android Studio 的 run 按钮运行很多次应用，Android Studio 很好地帮我们隐去了 APK 的生成流程，这中间经历了哪些流程，这里简单梳理记录下。

Android APK 本质上是一个压缩包，打开后会发现就是各种资源文件、一或多个 dex 文件、AndroidManifest.xml、resources.arsc 以及其他一些文件组成的。

Android 官网给出的构建流程图：

![](http://om9o63aks.bkt.clouddn.com/android_build.png)

**从图中可以总结为 7 个步骤**：

1、通过 aapt 打包 res 资源文件，生成 R.java、resources.arsc 和 res 文件（二进制 & 非二进制如 res/raw 和 pic 保持原样）

2、处理 .aidl 文件，生成对应的Java接口文件。

3、通过 Java Compiler 编译 R.java、Java 接口文件、Java 源文件，生成 .class 文件。

4、通过 dex 命令，将 .class 文件和第三方库中的 .class 文件处理生成 classes.dex。

5、通过 apkbuilder 工具，将 aapt 生成的 resources.arsc 和 res 文件、assets 文件和 classes.dex 一起打包生成apk。

6、通过 Jarsigner 工具，对上面的 apk 进行 debug 或 release 签名。

7、通过 zipalign 工具，将签名后的 apk 进行对齐处理。

更详细的流程图可以看下图：

![](http://om9o63aks.bkt.clouddn.com/android_build_detail.png)

**参考**

- [Android打包系列——打包流程梳理](http://mouxuejie.com/blog/2016-08-04/build-and-package-flow-introduction/)
- [Android逆向分析(2) APK的打包与安装](http://blog.zhaiyifan.cn/2016/02/13/android-reverse-2/) (非常详细)

