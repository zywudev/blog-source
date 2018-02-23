---
title: 解决 Genymotion 启动系统时出现「The virtual device got no IP address」问题
date: 2017-08-30 20:25:10
tags:
---

当首次启动 Genymotion 模拟器时出现如下错误，提示「The virtual device got no IP address」，根据提示的网址操作无果。

![](http://om9o63aks.bkt.clouddn.com/Genymotion-1.png)

其实在 VirtualBox 的启动窗口提示中，已经很明显了。这里需要 64 位 CPU，但是系统只检测到了 32 位 CPU。

![](http://om9o63aks.bkt.clouddn.com/Genymotion-2.png)

于是在设置中选择 64 位系统，发现找不到 `Ubuntu(64-bit)` 选项。

![](http://om9o63aks.bkt.clouddn.com/Genymotion-3.png)

这是由于在 BIOS 中未开启 VT（virtualization technology）导致，具体开启 VT 的方法详见 [https://bbs.yeshen.com/forum.php?mod=viewthread&tid=194](https://bbs.yeshen.com/forum.php?mod=viewthread&tid=194)

问题解决。







