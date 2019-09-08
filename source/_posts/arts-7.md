---
title: 每周 ARTS 第 7 期
date: 2019-09-07 21:41:01
tags: ARTS
---

## Algorithm

**描述**：

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

- 左括号必须用相同类型的右括号闭合。

- 左括号必须以正确的顺序闭合。

注意空字符串可被认为是有效字符串。

示例 1:

> 输入: "()"
>
> 输出: true

示例 2:

> 输入: "()[]{}"
>
> 输出: true

示例 3:

> 输入: "(]"
>
> 输出: false

示例 4:

> 输入: "([)]"
>
> 输出: false

示例 5:

> 输入: "{[]}"
>
> 输出: true

**思路**：

有效表达式的特点在于其子表达式应该也是有效表达式。

从整体表达式中一次删除一个较小的表达式，如果是一个有效的表达式，我们最后会留下一个空字符串。

采用栈结构解决这个问题。

**算法**：

1、初始化栈 S。

2、一次处理表达式的每个括号。

3、如果遇到开括号，我们只需将其推到栈上即可。这意味着我们将稍后处理它，让我们简单地转到前面的 子表达式。

4、如果我们遇到一个闭括号，那么我们检查栈顶的元素。如果栈顶的元素是一个相同类型的左括号，那么我们将它从栈中弹出并继续处理。否则，这意味着表达式无效。

5、如果到最后我们剩下的栈中仍然有元素，那么这意味着表达式无效。

```java
class Solution {
    
    private HashMap<Character, Character> mappings = new HashMap<Character, Character>();
    
    public Solution() {
         this.mappings.put(')', '(');
         this.mappings.put('}', '{');
         this.mappings.put(']', '[');
    }
    
    public boolean isValid(String s) {
        
        Stack<Character> stack = new Stack<Character>();

        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);

            // 当前是右括号
            if (this.mappings.containsKey(c)) {

                // 获取堆栈的顶部元素。如果堆栈是空的，设置一个哑值'#'
                char topElement = stack.empty() ? '#' : stack.pop();

                // 如果这个括号的映射与堆栈的顶部元素不匹配，则返回false
                if (topElement != this.mappings.get(c)) {
                  return false;
                }

            } else {
                // 如果它是一个左括号，推到堆栈
                stack.push(c);
             }
        }

        // 如果堆栈仍然包含元素，则它是一个无效表达式
        return stack.isEmpty();
    }
}
```

时间复杂度：$O(n)$。

空间复杂度：$O(n)$。

## Review

[Key habits and things I wish I knew earlier as a developer](https://medium.com/@rhamedy/key-habits-and-things-i-wish-i-knew-earlier-as-a-developer-43c9466a0407)

作为一名开发人员，我希望早点知道一些关键的习惯和事情

- 有效使用搜索引擎

- 使用现代化的 IDE 提高生产力

- 学习 Linux

- 学习 Git

- 自学 & 跟随技术趋势

- 精通至少一种编程语言

- 聚焦简约

- 加入社区/论坛

- 打造个人 IP

- 问，问，问

- 先把它写在纸上，然后转换成代码

- 从一开始就遵循样式指南、文档和编写测试

- 经常性解决难题和挑战

- 尽早开启白板编程

- 时间管理

- 保护个人隐私

- 追随那些能激励你的有影响力的人和公司

- 参加技术活动、研讨会、讲座和黑客马拉松

- 接受错误信息

- 选择正确类型的公司作为事业

- 先开发解决方案，再迭代完善

- 为自己 SEO

- 不要轻易放弃

- 不要拷贝项目

- 不要拖延

- 不要忽略其他学科

- 不要沉迷社交网络

- 不要失去希望

## Tip

在做应用启动、卡顿优化时，经常会用到 Android 性能分析工具 TraceView，这里简单介绍下 TraceView 的基础使用。

[Android 性能分析工具 TraceView](http://wuzhangyang.com/2019/09/04/android-traceview/)

## Share

**文章**：

1、[销售新套路，你中过招没](https://mp.weixin.qq.com/s/vhlK2PzwquqXNzKLp4yPYA)

各种销售套路，一文点醒你。

2、[机器人你好，人类再见---做人，别轻易放弃选择权](https://mp.weixin.qq.com/s/2mo4Zi8F7ip5dlb0AybSpw)

人类不是机器，不要丢了自由选择的权利。

**书籍**：

三表 2018 年出的书：

[我们只是讲道理](https://book.douban.com/subject/30275837/)

三表，工科出身，做过杂志主笔，当过建筑工人，干过广告，玩过创意。自媒体大潮兴起时，独立运营微信公众号“三表龙门阵”至今，以“犀利吐槽”为特色，辣评时事。

**电影**：

[美国工厂](https://movie.douban.com/subject/30390700/)

美国工厂 是一部 Netflix 原创纪录片，由 Higher Ground Productions 和 Participant Media 出品，荣获奥斯卡金像奖®提名并斩获艾美奖®的朱莉娅·赖克特和史蒂文·博格纳尔（《最后一辆车：通用王国的破产》《A Lion in the House》《正观“红色”》）打造。这部广受好评的电影深入研究了后工业时代的俄亥俄州，一位中国亿万富翁在当地一家废弃的通用汽车工厂中开设新工厂，并雇佣了 2000 名美国蓝领工人。随着高科技中国企业与美国工人阶级产生冲突，最初的希望和乐观遭受了挫折。

**软件**：

[Splashy](https://splashy.art/?ref=appinn)

一款跨平台的自动更换壁纸应用，基于高质量的 Unsplash 照片库，拥有 Windows、macOS、Linux 客户端，以及 Android 应用。

**言论**：

某公众号留言：

> 大家955的时候，有些聪明人觉得如果自己996那么裁掉的就是别人了。结果聪明人太多了…… 

飞总：

> 社会学上有一个很有意思的现象。当大家都老老实实守规矩的时候，如果有人不守规矩的话，这些不守规矩的人是要获利的。比如说大家结成价格联盟，这个时候有人突然闪电大降价，这位就可以卖掉很多，从而获利。如果大家都坚持工作8个小时，有人突然工作12个小时了，这些人的产出就会立刻可以获利。

caoz ：

> 赚自己真的看得懂的钱，做自己真的有把握的事情。

**其他**：

[沙文主义](https://zh.wikipedia.org/wiki/%E6%B2%99%E6%96%87%E4%B8%BB%E4%B9%89)：盲目热爱自己所处的团体，并经常对其他团体怀有恶意与仇恨。





