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

