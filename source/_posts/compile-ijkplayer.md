---
title: ijkplayer 编译实践
date: 2019-10-14 16:41:32
tags:
- ijkplayer
- Android音视频
---

## 编译环境

### Linux 环境

由于主机是 Windows 系统，所以使用 VMware 安装了 Ubuntu 系统。

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
sudo apt-get install git
sudo apt-get install yasm
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

默认为最少支持, 如果足够你使用, 可以跳过这一步. 否则可以改为以下配置:

- `module-default.sh` 更多的编解码器/格式
- `module-lite-hevc.sh` 较少的编解码器/格式(包括hevc)
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



### 编译 ffmpeg

### 编译 ijkplayer

