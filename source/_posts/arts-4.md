---
title: 每周 ARTS 第 4 期
date: 2019-08-04 20:36:52
tags: ARTS
---

## Algorithm

#### 罗马数字转整数

**描述**

罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。

```java
字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```


例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

- I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
- X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
- C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。

给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。

示例 1:

```java
输入: "III"
输出: 3
```


示例 2:

```java
输入: "IV"
输出: 4
```

示例 3:

```java
输入: "IX"
输出: 9
```

示例 4:

```java
输入: "LVIII"
输出: 58
解释: L = 50, V= 5, III = 3.
```


示例 5:

```java
输入: "MCMXCIV"
输出: 1994
解释: M = 1000, CM = 900, XC = 90, IV = 4.
```

**方法**

1\. 首先将所有的组合可能性列出并添加到哈希表中

2\. 然后对字符串进行遍历，由于组合只有两种，一种是 1 个字符，一种是 2 个字符，其中 2 个字符优先于 1 个字符。

3\. 先判断两个字符的组合在哈希表中是否存在，存在则将值取出加到结果 ans 中，并向后移2个字符。不存在则将判断当前 1 个字符是否存在，存在则将值取出加到结果 ans 中，并向后移 1 个字符。

4\. 遍历结束返回结果 ans。

```java
public int romanToInt(String s) {
    Map<String, Integer> map = new HashMap<>();
    map.put("I", 1);
    map.put("IV", 4);
    map.put("V", 5);
    map.put("IX", 9);
    map.put("X", 10);
    map.put("XL", 40);
    map.put("L", 50);
    map.put("XC", 90);
    map.put("C", 100);
    map.put("CD", 400);
    map.put("D", 500);
    map.put("CM", 900);
    map.put("M", 1000);

    int ans = 0;
    for(int i = 0;i < s.length();) {
        if(i + 1 < s.length() && map.containsKey(s.substring(i, i+2))) {
            ans += map.get(s.substring(i, i+2));
            i += 2;
        } else {
            ans += map.get(s.substring(i, i+1));
            i ++;
        }
    }
    return ans;
}
```

## Review

[How to Ensure Your Software Projects Actually Finish](https://medium.com/better-programming/how-to-ensure-your-software-projects-actually-finish-5e0dd7ea46fc)

关于如何保证项目正常完成，不至于烂尾。作者分享了几种方法。

1\. 及时清理障碍

项目中随时都可能出现障碍，障碍的出现会影响到项目的进度、挫败团队的动力。所以要尽快清理障碍，可以指派一个团队来负责这些障碍，并确保定期与团队跟进，让他们知道工作的重要性。

2\. 创建里程碑

设立里程碑，这样 PM 能够了解团队和个人的工作能力，从而更好的安排未来的工作。虽然里程碑让人感到有压力，但确实是很有必要的。

3\. 沟通是关键

沟通的重要性不言而喻。

4\. 明确你的目标

技术项目没有终点。需求永不停歇。

确保有一个一致同意的目标，确保产品的第一个版本是重要和必要的。

## Tip

学习了 MediaExtractor 和 MediaMuxer 的使用：

[ 使用 MediaExtractor 和 MediaMuxer 分离合成音视频](http://wuzhangyang.com/2019/08/01/android-mediamuxer-and-mediaextractor/)

## Share

推荐一下我一直在听的一个播客节目吧，它叫 「故事FM」。

> 故事FM 是一档普通人自述亲身经历的播客。我们以制作电影的方式，用你的声音，讲述你的故事。每周一、三、五更新。公号：故事 FM。故事征集：http://cn.mikecrm.com/v1w20un

很多有意思的故事，都是真人真事，听多了你会发现，原来每个人的活法都不一样，每个人的人生都很精彩。

网易云音乐、蜻蜓FM、喜马拉雅等都能收听，强烈推荐。



