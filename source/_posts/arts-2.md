---
title: 每周 ARTS 第 2 期
date: 2019-07-21 21:59:59
tags: ARTS
---

## Algorithm

### 整数反转

**描述**

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

**示例**

```java
输入: 123
输出: 321
```

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−2<sup>31</sup>,  2<sup>31</sup> − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

**方法**

求余运算即可得到最后一位数，但是在拼凑成反转整数时需要考虑到整数溢出问题。

```java
public int reverse(int x) {
    int rev = 0;
    while (x != 0) {
        int pop = x % 10;
        x /= 10;
        if (rev > Integer.MAX_VALUE/10 || (rev == Integer.MAX_VALUE / 10 && pop > 7)) return 0;
        if (rev < Integer.MIN_VALUE/10 || (rev == Integer.MIN_VALUE / 10 && pop < -8)) return 0;
        rev = rev * 10 + pop;
    }
    return rev;
}
```

**复杂度分析**

时间复杂度： O(log(x))

空间复杂度：O(1)

## Review

无。

## Tip

记录了一篇 Camera 的基本使用。

[使用 Camera 和 Camera2 采集视频数据](http://wuzhangyang.com/2019/07/15/Android-Media-Notes-4-Camera/)

## Share

前几天网上又看到一起开门杀，河北一女子随意开车门致使电动车人死亡。这样的事件经常发生，如果每个人开车门的时候有意识多观察后方，这样的事件就不可能发生。因此特地写一篇文章分享下我知道的两种常用方法，有效避免类似事件发生，欢迎转发，让更多人知道。

[又见开门杀](https://mp.weixin.qq.com/s/-W0nAWnXz6p5wMF5azYWYA)