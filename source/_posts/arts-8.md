---
title: 每周 ARTS 第 8 期
date: 2019-09-15 11:21:54
tags: ARTS
---

## Algorithm

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

**示例：**

```java
输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
```

**方法1**：

迭代算法。

设定一个哨兵节点 `prehead`，便于最后返回合并后的链表。

维护一个 `prev` 指针，迭代中调整它的 `next` 指针。

不断遍历两个链表，直到 `l1` 或者 `l2` 指向了 `null`，如果 `l1` 当前位置的值小于等于 `l2`，就把 `l1` 的值接到 `prev` 节点的后面同时将 `l1` 的指针往后移一个，否则对 `l2` 做同样的操作。

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode prehead = new ListNode(-1);

        ListNode prev = prehead;
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                prev.next = l1;
                l1 = l1.next;
            } else {
                prev.next = l2;
                l2 = l2.next;
            }
            prev = prev.next;
        }

        prev.next = l1 == null ? l2 : l1;

        return prehead.next;
    }
}
```

时间复杂度 : $O(m+n)$

空间复杂度：$O(1)$

**方法2**：

递归算法。

每次都是两个链表头部较小的一个与剩下元素的合并操作。

```java
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        else if (l2 == null) {
            return l1;
        }
        else if (l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next, l2);
            return l1;
        }
        else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }

    }
}
```

时间复杂度 : $O(m+n)$

空间复杂度：$O(m+n)$

## Review

[Ace your first year as a junior developer with this advice](https://www.freecodecamp.org/news/ace-your-first-year-as-a-junior-developer-with-this-advice-bbc68b6fe2d9/)

给初级程序员的职场建议 ： 

- 知识技能有缺口是没问题的，重要的是不断学习

- 问问题是好事，要积极寻求帮助

- 代码评审是你的朋友

- 把大的任务分解成小任务

- 保持简洁：走通、重构、优化。

- 学习如何编写整洁代码

- 学习如何阅读代码

## Tip

应用启动时间的长短，影响到用户体验。对研发人员来说，启动速度是我们的“门面”。

- 应用启动有哪些流程？

- 如何检测应用启动耗时操作？

- 如何优化应用启动速度？

这篇文章给你答案。

[Android 应用启动速度优化](http://wuzhangyang.com/2019/09/09/android-launch-time-performance-optimization/)

## Share

**文章**：

1、[二十出头，老气横秋](https://mp.weixin.qq.com/s/mX8aphHjd-LshjjuP0s5uQ)

2、[不思考的恶](https://mp.weixin.qq.com/s/W9NYqhHddtnV7h-_NbspJQ)

3、[再无风清扬，再有少年郎](https://mp.weixin.qq.com/s/NG1tbz6N-yC9RgGCElSdpg)

**书籍**：

[棋王](https://book.douban.com/subject/1020961/)

> 在处女作《棋王》中，阿城表现出自己的哲学：“普遍认为很苦的知青生活，在生活水准低下的贫民阶层看来，也许是物质上升了一级呢！另外就是普通人的‘英雄’行为常常是历史的缩影。那些普通人在一种被迫的情况下，焕发出一定的光彩。之后，普通人又复归为普通人，并且常常被自己有过的行为所惊吓，因此，从个人来说，常常是从零开始，复归为零，而历史由此便进一步。” 阿城笔下著名的"棋王"王一生是近世以来罕见的一个深刻体现了道家文化特征的人物形象。王一生深得老子的阴柔之气。他的性格是坚忍而沉着的。《棋王》表面上写棋，实质上则具有多层次的象征意义，表现着他对中国文化传统的历史评价和对中国文化进步的展望。

**纪录片**：

[河西走廊](https://movie.douban.com/subject/24736278/)

> 《河西走廊》是一部由中共甘肃省委宣传部、中央电视台科教频道联合出品，北京伯璟文化传播有限公司承制的十集系列纪录片。该片以位于中国西部的重要通道，丝绸之路的黄金段——河西走廊为讲述对象，从政治、军事、经济、文化、宗教等多角度呈现了从汉代开始直至今天，河西走廊及其连接的中国 西部 的历史，以及它对中国历史和文明进程中所发挥的独特作用。“河西走廊关乎国家经略”是贯穿全篇的主题。

**软件**：

[Notebook](https://www.zoho.com/notebook/)

Notebook 是由 Zoho 公司推出的完全免费的资料收集与整理的[笔记类工具，可用于记笔记、创建任务清单、记录语音与视频等。Zoho Notebook 非常注重设计和细节，笔记以卡片形式显示，颜值很高，与 Google Keep 有点相似。笔记支持跨平台跨设备云端同步，无论在何处何地，都能给你带来简单、轻松、高效的笔记体验。

**言论**：

稻盛和夫：

> 我们的人生是由命运和因果报应两条法则互相交织而成的。这两者互相干涉，比如当命运非常恶劣时，做一点好事，并不会出现好的结果，因为仅有的一点善行为强势的厄运所淹没。同样，当好运非常旺盛时，稍稍做点坏事，也不会马上出现恶因招恶果的情形。 

Martin Fowler：

> 任何傻瓜都能写计算机能理解的代码，优秀的程序员编写人类能够理解的代码

