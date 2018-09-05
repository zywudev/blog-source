---
title: 推荐一个好用的 Android 屏幕适配的插件
date: 2018-04-18 22:15:56
tags:
---

Android 屏幕适配一直是个消耗时间，但没啥意思的活。

关于 Android 屏幕适配的基本知识可以参考这篇文章：

- [Android 屏幕适配笔记](http://wuzhangyang.com/2017/08/02/android-screen-adaptation/)  

今天推荐一种适配方案，能够节约大部分适配时间。

## dp 适配

我们知道获取屏幕最小宽度 dp 值的方法：

```java
DisplayMetrics dm = new DisplayMetrics();

getWindowManager().getDefaultDisplay().getMetrics(dm);

int widthPixels = dm.widthPixels;   

float density = dm.density;

float widthDP = widthPixels / density;
```

而 dp 适配的原理就是根据 dp 值等比缩放。根据**“最小宽度（Smallest-width）限定符”**，即如果当前设备最小宽度（以 dp 为单位）为 400dp，那么系统会自动找到对应的 values-sw400dp 文件夹下的 dimens.xml 文件。

所以如果可以以某一 widthDP 为基准，生成所有设备对应的  dimens.xml 文件，就可以省去很多适配时间了。

网上已经有大神写了一个自动生成的插件，非常好用。下面介绍下使用方法。

1、首先是下载 ScreenMatch 插件，这个不用多讲了，直接在 Android Studio 中下载即可，安装完成重启 AS。

2、在项目的默认 values 文件夹中需要提供一份 dimens.xml 文件。这个文件中的 dp 值和 sp 值就是基准值。写法如下。

```jav
<?xml version="1.0" encoding="UTF-8"?>
<resources>

    <!-- Your custom size defind by references, can be writted in anywhere, any module, any values/*.xml, for example: -->
    <dimen name="card_common_margin_left">@dimen/dp_15</dimen>

    <!-- dp and sp values, must be defind in this file! -->
    <!-- view size,you can add if there is no one -->
    <dimen name="dp_m_60">-60dp</dimen>
    <dimen name="dp_m_30">-30dp</dimen>
    <dimen name="dp_m_20">-20dp</dimen>
    <dimen name="dp_m_12">-12dp</dimen>
    <dimen name="dp_m_10">-10dp</dimen>
    <dimen name="dp_m_8">-8dp</dimen>
    <dimen name="dp_m_5">-5dp</dimen>
    <dimen name="dp_m_2">-2dp</dimen>
    <dimen name="dp_m_1">-1dp</dimen>
    <dimen name="dp_0">0dp</dimen>
    <dimen name="dp_0_1">0.1dp</dimen>
    <dimen name="dp_0_5">0.5dp</dimen>
    <dimen name="dp_1">1dp</dimen>
    <dimen name="dp_1_5">1.5dp</dimen>
    <dimen name="dp_2">2dp</dimen>
 
    <!-- font size,you can add if there is no one -->
    <dimen name="sp_6">6sp</dimen>
    <dimen name="sp_7">7sp</dimen>

</resources>
```

3、在任何目录下右键选择 ScreenMatch 选项，执行自动生成其他 widthDP 设备对应的 dimens.xml 文件。

4、插件默认的 widthDP 基准值是 360dp，一般手机的 widthDP 都是 360dp。这个插件会自动生成  widthDP 为 384,392,400,410,411,480,533,592,600,640,662,720,768,800,811,820,960,961,1024,1280,1365 下的 dimen 文件。

5、当然你也可以通过修改以下配置文件的属性来满足自己的需求。记得修改配置后，需要删除自动生成的文件夹，重新点击 ScreenMatch 生成。

```java
############################################################################
# Start with '#' is annotate.                                              #
# In front of '=' is key, cannot be modified.                              #
# More information to visit:                                               #
#   http://blog.csdn.net/fesdgasdgasdg/article/details/52325590            #
#   http://download.csdn.net/detail/fesdgasdgasdg/9913744                  #
#   https://github.com/mengzhinan/PhoneScreenMatch                         #
############################################################################
#
# You need to refresh or reopen the project every time you modify the configuration,
# or you can't get the latest configuration parameters.
#
#############################################################################
#
# Base dp value for screen match. Cut the screen into [base_dp] parts.
# Data type is double. System default value is 360.
# I advise you not to modify the value, be careful !!!!!!!!! _^_  *_*
base_dp=360            // widthDP 默认基准
# Also need to match the phone screen of [match_dp].
# If you have another dp values.
# System default values is 384,392,400,410,411,480,533,592,600,640,662,720,768,800,811,820,960,961,1024,1280,1365
match_dp=    // 可以添加你想要的 widthDP 值
# If you not wanna to match dp values above. Write some above values here, append value with "," .
# For example: 811,961,1365 
ignore_dp=     // 添加不需要生成的 widthDP 值
# They're not android module name. If has more，split with , Symbol.
# If you set, it will not show in SelectDialog.
# If you have, write here and append value with "," .
# For example: testLibrary,commonModule
# System default values is .gradle, gradle, .idea, build, .git
ignore_module_name=
# Use which module under the values/dimen.xml file to do the base file,
# and generated dimen.xml file store in this module?
# Default value is 'app'.
match_module=app
# Don't show select dialog again when use this plugin.
# System screen match will use the last selected module name or default module name.
# You can give value true or false. Default value is false.
not_show_dialog=false
# Do you want to generate the default example dimens.xml file?
# In path of .../projectName/screenMatch_example_dimens.xml, It does not affect your project code.
# You can give value true or false. Default value is false.
not_create_default_dimens=false
# Does the font scale the same size as the DP? May not be accuracy.
# You can give value true or false. Default value is true. Also need scaled.
is_match_font_sp=true
# Do you want to create values-wXXXdp folder or values-swXXXdp folder ?
# I suggest you create values-swXXXdp folder,
# because I had a problem when I was working on the horizontal screen adapter.
# values-swXXXdp folder can solve my problem.
# If you want create values-swXXXdp folder, set "create_values_sw_folder=true",
# otherwise set "create_values_sw_folder=true".
# Default values is true.
create_values_sw_folder=true

```

6、对于其他 module， 只需要在 values 文件夹下有一套与 app module 一样的 dimens 文件即可达到适配。

具体的插件原理以及更多细节可以直接去看插件作者的文章 [Android dp方式的屏幕适配工具使用(Android Studio插件方式)](https://blog.csdn.net/fesdgasdgasdg/article/details/78108169)，非常感谢作者。





