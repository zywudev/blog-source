---
title: Java 对象的自我救赎
date: 2019-01-14 16:17:23
tags:
---

JVM 通过[可达性分析算法判断一个对象是否可以被回收](http://wuzhangyang.com/2019/01/12/how-do-jvm-kown-if-an-object-can-be-recycled/) ，但并不是一个对象不可达时，就宣告“死刑”的，此时只是暂时处于”缓刑“阶段。要宣告一个对象“死刑”，至少还要经历两次标记过程。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/java_object_self_redemption.png)



没有必要执行 `finalize()` 方法的筛选条件取决于：

1、 `finalize()` 方法已经被执行过（finalize()`只会执行一次）。

2、对象没有重写 `finalize()`方法。 

如果一个对象有必要执行  `finalize()` 方法，会进入 F-Queue 队列，等待 Finalizer 线程执行。

**因此如果想要完成对象自救， `finalize()`是逃脱死亡的最后一次机会，重新与引用链上的任何一个对象关联起来就可以，在第二次标记时，对象会被移出回收队列，完成自救。**

```java
public class FinalizeEscapeGC {

    public static FinalizeEscapeGC SAVE_HOOK = null;

    public static void main(String[] args) throws InterruptedException {
        SAVE_HOOK = new FinalizeEscapeGC();
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("我挂了");
        }
    }

    public void isAlive() {
        System.out.println("我还活着");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("执行 finalize 方法");
        // 把当前对象( this )赋值给某个类变量, 重新与引用链建立引用
        SAVE_HOOK = this;
    }
}
```

扩展：

 `finalize()` 方法的执行线程 Finalizer 优先级级别低，无法保证  `finalize()` 方法什么时候执行，执行是否符合预期，使用不当会影响性能。

Java 9 中已经将  `finalize()` 方法标记为废弃了，如果没有特别的原因，不要重写  `finalize()` 方法，也别指望它能回收资源。相反，尽量使用 `try-finally` 、 `try-with-resources` 等机制是非常好的资源回收方法。