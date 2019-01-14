---
title: Java 对象的自我救赎
date: 2019-01-14 16:17:23
tags:
---

JVM 通过[可达性分析算法判断一个对象是否可以被回收](http://wuzhangyang.com/2019/01/12/how-do-jvm-kown-if-an-object-can-be-recycled/) ，但并不是一个对象不可达时，就宣告“死刑”的，此时只是暂时处于”缓刑“阶段，要宣告一个对象“死刑”，至少还要经历两次标记过程。

