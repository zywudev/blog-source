---
title: ijkplayer 编译实践
date: 2019-10-14 16:41:32
tags: Android音视频
---

记录 ijkplayer 的编译过程，以及遇到的问题，有需要的朋友可以参考。

## 编译环境

### Linux 环境

由于主机是 Windows 系统，所以使用 VMware 安装了 Ubuntu 18.0.4 系统。

VMware 安装 Ubuntu 系统的安装步骤网上非常多，这篇文章比较详细，没有经验的可以参考。

https://zhuanlan.zhihu.com/p/38797088

### Android SDK

下载地址：https://developer.android.com/studio#downloads

### Android NDK

下载地址：https://developer.android.google.cn/ndk/downloads/older_releases.html

注意 NDK 的最小版本支持是 10e，目前不支持 NDK 15！我这边下载的是 `android-ndk-r14b`。

Android SDK 和 Android NDK 下载解压后，需要配置环境变量，可以参考我写的这篇文章。

http://wuzhangyang.com/2019/10/14/ubuntu-android-studio/

### 安装 git 和 yasm

```shell
sudo apt install git
sudo apt install yasm
```

注意，如果安装报错，先要执行 `sudo apt update` 进行更新。

## 开始编译

### 拉取 ijkplayer 源码

```shell
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
cd ijkplayer-android
git checkout -B latest k0.8.8
```

### 初始化 android

```shell
./init-android.sh
```

### 初始化 openssl 支持 https

```shell
./init-android-openssl.sh
```

### 配置编解码器格式支持

默认为最少支持，如果足够你使用，可以跳过这一步，否则可以改为以下配置:

- `module-default.sh` 更多的编解码器/格式

- `module-lite-hevc.sh` 较少的编解码器/格式(包括 hevc)
- `module-lite.sh` 较少的编解码器/格式(默认情况)

```shell
# 进入 config 目录
cd config

# 删除当前的 module.sh 文件
rm module.sh

# 创建软链接 module.sh 指向 module-default.sh
ln -s module-default.sh module.sh
```

### 编译 openssl

```shell
# 进入 android/contrib 目录
cd android/contrib

# 清除 openssl 的编译文件
./compile-openssl.sh clean

# 编译 openssl
./compile-openssl.sh all
```

`./compile-openssl.sh` 后跟 `all` 表示编译所有 CPU 架构的 so 库， 如果只编译指定 CPU 架构的 so 库，后面就跟 CPU 架构，比如：` ./compile-ffmpeg.sh armv7a `。

这里，在执行 `./compile-openssl.sh all` 时出现了编译错误：**ERROR: Failed to create toolchain.**

```shell
jaqen@jaqen-virtual-machine:~/Android/Projects/ijkplayer-android/android/contrib$ ./compile-openssl.sh all
====================
[*] check archs
====================
FF_ALL_ARCHS = armv5 armv7a arm64 x86 x86_64
FF_ACT_ARCHS = armv5 armv7a arm64 x86 x86_64

--------------------
[*] make NDK standalone toolchain
--------------------
build on Linux x86_64
ANDROID_NDK=/home/jaqen/Android/Sdk/android-ndk-r14b
IJK_NDK_REL=14.1.3816874
NDKr14.1.3816874 detected

--------------------
[*] make NDK standalone toolchain
--------------------
build on Linux x86_64
ANDROID_NDK=/home/jaqen/Android/Sdk/android-ndk-r14b
IJK_NDK_REL=14.1.3816874
NDKr14.1.3816874 detected
HOST_OS=linux
HOST_EXE=
HOST_ARCH=x86_64
HOST_TAG=linux-x86_64
HOST_NUM_CPUS=4
BUILD_NUM_CPUS=8
Auto-config: --arch=arm
ERROR: Failed to create toolchain.

```

解决办法是安装 python 后再执行编译。

```shell
sudo apt install python
```

### 编译 ffmpeg

```shell
# 清除 ffmpeg 的编译文件
./compile-ffmpeg.sh clean

# 编译 ffmpeg
./compile-ffmpeg.sh all
```

执行 `./compile-ffmpeg.sh all` 时出现编译错误：**linux/perf_event.h: No such file or directory**。

解决办法是在 `config` 文件夹下的 `module.sh` 文件中加入下面两句：

```shell
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-linux-perf"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-bzlib"
```

再重新执行编译。

### 编译 ijkplayer

```shell
# 进入 android 目录
cd ..

# 编译 ijkplayer
./compile-ijk.sh all
```

 编译完成之后，在 `android/ijkpleyer` 文件夹的对应架构文件下，在`/src/main/libs/架构名/`下生成`libijkplayer.so`、`libijkffmpeg.so`、`libijksdl.so` 三个文件。 

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/ijkplayer_so.png)

至此，ijkplayer 的编译工作就全部完成了。

编译过程中遇到问题的朋友欢迎留言交流。

不想编译的朋友，可以在公众号 「JaqenAndroid」后台回复 `ijk` 获取 so 包。

