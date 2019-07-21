---
title: 每周 ARTS 第 1 期
date: 2019-07-14 16:33:31
tags: ARTS
---

## Algorithm

### 1. 两数之和

**描述**

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

**示例**

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9，所以返回 [0, 1]

**方法**

首先想到可能就是暴力法，遍历每个元素，查找是否存在一个值与 target - x 相等的目标元素。但是时间复杂度 为 O(n<sup>2</sup>)。所以可以考虑以空间换时间的方式。

采用哈希表。

一次迭代，将元素插入到哈希表，同时检测表中是否存在当前元素对应的目标元素。如果存在就找到了对应解。

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for(int i = 0; i < nums.length; i++) {
        int value = target - nums[i];
        if(map.containsKey(value)) {
            return new int[]{map.get(value),i};
        }
        map.put(nums[i],i);
    }
    throw new IllegalArgumentException("No two sum solution");
}
```

**复杂度分析**

时间复杂度：O(n)

空间复杂度：O(n)

## Review

[Technology Trends That Will Dominate 2019: Big Data, IoT, AWS and AI](https://medium.com/datadriveninvestor/technology-trends-that-will-dominate-2019-big-data-iot-aws-and-ai-b721a62941fc)

2019 年将占据主导地位的技术趋势：大数据、物联网、AWS 和人工智能

作者在文章中主要讨论了 2019 年较为明显的技术趋势。

1. 区块链
2. 物联网和智能家居
3. 人工智能和机器学习
4. 软件定义安全
5. 自动化
6. 增强现实和虚拟现实
7. 点播业务
8. 智能应用
9. 人性化大数据(可视化、移情、定性)

作者并没有就每个技术趋势展开讨论，只是简单列举了以上几点技术趋势。

不得不承认，大部分都是当前以及未来十年二十年的发展趋势，各大互联网公司都在加大投入。如果说 2010 年到 2020 年这十年是互联网的时代，未来十年一定是物联网的时代，尤其是 5G 技术的成熟和普及应用。

## Tip

一直对 Android 音视频技术有兴趣。之前只是了解一点点，这周从基础开始学习音视频技术。学习总结均以博客文章的形式记录。

1. [使用三种不同的方式绘制图片](http://wuzhangyang.com/2019/07/05/Android-Media-Notes-1-Draw-Image/)
2. [使用 AudioRecord 采集音频](http://wuzhangyang.com/2019/07/08/Android-Media-Notes-2-AudioRecord/)
3. [使用 AudioTrack 播放 PCM 音频](http://wuzhangyang.com/2019/07/08/Android-Media-Notes-3-AudioTrack/)

## Share

网上看到杭州的章子欣小朋友的事，心情比较难受。

这里不是蹭热点，分享一个对大众非常有用的公安部失踪儿童找回系统，可能很多人还不知道。所以为了让更多人知道，特地写了一篇文章介绍了下，希望看到的朋友奔走相告。

文章在这里：「[团圆系统](http://mp.weixin.qq.com/s?__biz=MzUyOTA2MjA2NQ==&mid=2247483846&idx=1&sn=bcda32fff95c4500320b2f5e720be23f&chksm=fa678018cd10090e51eee4059e4ff486b3eff310d87e9de198c1e55f4be16201c65f4b61303c&token=969564947&lang=zh_CN#rd)」

