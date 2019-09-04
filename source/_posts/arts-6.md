---
title: 每周 ARTS 第 6 期
date: 2019-09-1 20:24:16
tags: ARTS
---

## Algorithm

**描述**

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 ""。

**示例 1**:

输入: ["flower","flow","flight"]

输出: "fl"

**示例 2**:

输入: ["dog","racecar","car"]

输出: ""

解释: 输入不存在公共前缀。

**说明:**

所有输入只包含小写字母 a-z 。

**解法 1：**

LCP(S1...Sn) == LCP(LCP(LCP(S1,S2),S3),...Sn)

依次遍历字符串数组[S1,...,Sn], 当遍历到第 i 个字符串，找到最长公共字符前缀 LCP(S1,...Si)。

```java
public String longestCommonPrefix(String[] strs) {
    if(strs.length == 0) return "";
    String prefix = strs[0];
    for(int i = 1; i < strs.length; i++) {
        while(strs[i].indexOf(prefix) != 0) {
            prefix = strs[0].substring(0,prefix.length() - 1);
            if (prefix.isEmpty()) return "";
        }
    }
    return prefix;
}
```

时间复杂度：O(S)，S 是所有字符串中字符数量的总和。

空间复杂度：O(1)。

**解法 2：**

从前到后对比每个字符串相同列上的字符。

```java
public String longestCommonPrefix(String[] strs) {
    if (strs == null || strs.length == 0) return "";
    for (int i = 0; i < strs[0].length() ; i++){
        char c = strs[0].charAt(i);
        for (int j = 1; j < strs.length; j ++) {
            if (i == strs[j].length() || strs[j].charAt(i) != c)
                return strs[0].substring(0, i);             
        }
    }
    return strs[0];
}
```

时间复杂度：$O(S)$，S 是所有字符串中字符数量的总和。

空间复杂度：$O(1)$。

**解法 3:**

分治思想。

$LCP(S1...Sn) == LCP(LCP(S1...Sk）,LCP(Sk+1...Sn))$

```java
public String longestCommonPrefix(String[] strs) {
    if (strs == null || strs.length == 0) return "";    
        return longestCommonPrefix(strs, 0 , strs.length - 1);
}

private String longestCommonPrefix(String[] strs, int l, int r) {
    if (l == r) {
        return strs[l];
    }
    else {
        int mid = (l + r)/2;
        String lcpLeft =   longestCommonPrefix(strs, l , mid);
        String lcpRight =  longestCommonPrefix(strs, mid + 1,r);
        return commonPrefix(lcpLeft, lcpRight);
   }
}

String commonPrefix(String left,String right) {
    int min = Math.min(left.length(), right.length());       
    for (int i = 0; i < min; i++) {
        if ( left.charAt(i) != right.charAt(i) )
            return left.substring(0, i);
    }
    return left.substring(0, min);
}
```

时间复杂度：O(S)，S 是所有字符串中字符数量的总和。

空间复杂度：O（m⋅log(n))

## Review

无

## Tip

很多人，在学生阶段习惯了被动、填鸭式学习，进入社会后，不知道如何去自我学习，严重限制了提升自己的知识和技能的机会。

在这个飞速发展的世界里，做一个终身学习者，自我学习的能力越强越好。

最近看了一本书《软技能: 代码之外的生存指南》，里面的一个主题，讲述了技术人员如何在新技术发展日新月异的世界里，如何快速高效学习技术，以下是我整理的「十步学习法」笔记，供大家参考。

[程序员成长离不开的技能](https://mp.weixin.qq.com/s/nmcU3dWGYu-mfewl7IbPfg)

## Share

[没有副业的人，太难了。。。](https://mp.weixin.qq.com/s/PMG6U5IM5-azv5dlZNDlHA)

[炒鞋，是挣钱新门路还是新一轮割韭菜？](https://mp.weixin.qq.com/s/6a98PnMCHnB_jfcR7GTz-w)