---
title: YUV 格式详解
date: 2018-12-10 20:19:23
tags: Android
toc: true
---

## 什么是 YUV/YCbCr/YPbPr？

一种颜色编码方法，主要用于电视系统以及模拟视频领域，将亮度信息与色彩信息分离，解决了彩色电视与黑白电视的兼容问题。

Y：明亮度（Luminance、Luma），即灰度值。

U、V、Cb、Cr、Pb、Pr：色度（Chrominance或Chroma），描述影像色彩和饱和度。

YUV码流的存储格式与采样方式密切相关，主流采样方式有：YUV4:4:4、YUV4:2:2、YUV4:2:0。

## 什么是 4:4:4、4:2:2、4:2:0？

人眼对色度的敏感程度低于对亮度的敏感程度。主要原因是视网膜杆细胞多于视网膜锥细胞，其中视网膜杆细胞的作用就是识别亮度，视网膜锥细胞的作用就是识别色度。所以，眼睛对于亮度的分辨要比对颜色的分辨精细一些。

因此，视频存储时没有必要存储全部颜色信号，适当降采样可以节省存储空间。

如下图，亮度信号都是全分辨率存储，色度信号并不是用全分辨率存储。

- YUV444 格式中每 4 点 Y 采样，有 4 点 U 和 4 点 V。即每个像素的 YUV 的数据都会存放起来。
- YUV422 格式中每 4 点 Y 采样，有 2 点 U 和 2 点 V。色度信号的扫描线数量与亮度信号一样，但是每条扫描线上的色度采样点数只有亮度信号的一半。当 422 信号被解码时，缺失的色度采样，通常由一定的内插补点算法补充。
- YUV420 格式是色度分辨率最低的格式，也是 DVD 所使用的格式。无论横线还是纵向，色度信号的分辨率都只有亮度信号的一半。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/yuv-format.png)



## YUV 两种存储格式

- planar：先连续存储所有像素点的 Y，紧接着存储所有像素点的 U，随后是所有像素点的 V。
- packed：每个像素点的 Y,U,V 是连续交错存储的。

假设一个分辨率为 8 X 4 的 YUV 图像，它们的存储格式如下图：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/yuv420p.png)



![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/yuv420sp.png)



