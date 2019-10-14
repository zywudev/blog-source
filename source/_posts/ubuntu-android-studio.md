---
title: Ubuntu 下 Android Studio 安装与配置
date: 2019-10-14 14:12:28
tags: Android Studio
---

本文主要记录一下 Ubuntu 环境下 Android Studio 的安装和配置，方便查找使用。

## Android Studio 安装

Android Studio 下载地址：

https://developer.android.com/studio

下载后，直接解压即可。

然后进入 `/android-studio/bin/` 文件下，会看到一个`studio.sh`的执行文件。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/ubuntu_android_studio1.png)

终端命令进入 `/android-studio/bin/ ` , 执行命令 `./studio.sh` ，进行安装即可，没啥好说的。

```shell
cd android-studio/bin
./studio.sh
```

 ## Android Studio 配置

### 配置 Android 环境变量

终端命令：

```shell
sudo gedit ~/.bashrc
```

在文档末尾添加：

```shell
# 配置 Android 环境变量
# 你的ADB路径
ADB=/home/jaqen/Android/Sdk/platform-tools
export ADB
# 你的ANDROID_NDK和ANDROID_SDK 路径
ANDROID_NDK=/home/jaqen/Android/Sdk/android-ndk-r14b
export ANDROID_NDK
ANDROID_SDK=/home/jaqen/Android/Sdk
export ANDROID_SDK 
# 加入到PATH路径
PATH=${PATH}:${ADB}:${ANDROID_NDK}:${ANDROID_SDK}
```

NDK 需要单独去下载：

https://developer.android.google.cn/ndk/downloads/older_releases.html

注意需要执行下面的命令，使修改生效。

```shell
source ~/.bashrc
```

### 设置 Android Studio 别名

```shell
# 你的android studio安装路径
alias as=/home/jaqen/Downloads/android-studio-ide-191.5900203-linux/android-studio/bin/studio.sh
```

然后，就可以用命令行执行命令 `as` 启动 Android Studio 了。

### 制作启动图标

制作一个启动图标，方便鼠标点击启动。

终端命令进入 `/android-studio/bin/ ` ，新建一个 `Studio.desktop` 文件，然后打开。

```shell
gedit Studio.desktop
```

输入以下内容：

```shell
[Desktop Entry]
Name=AndroidStudio
Type=Application
# 你的Android Studio 图标路径
Icon=/home/jaqen/Downloads/android-studio-ide-191.5900203-linux/android-studio/bin/studio.png
# 你的Android Studio 启动路径
Exec=sh /home/jaqen/Downloads/android-studio-ide-191.5900203-linux/android-studio/bin/studio.sh
```

保存。

右键该文件 > 属性 > 权限 > 选择允许作为程序执行文件，这个时候文件名和图标都会相应改变了。

好了，双击可以启动 Android Studio 了。

将启动图标添加到 `/usr/share/applications/`下。

```shell
sudo mv /home/jaqen/Downloads/android-studio-ide-191.5900203-linux/android-studio/bin/Studio.desktop /usr/share/applications
```

终端命令：

```shell
nautilus  usr/share/applications
```

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/ubuntu_android_studio2.png)



