---
title: Java 对象的自我救赎
date: 2019-01-14 16:17:23
tags:
---

JVM 通过[可达性分析算法判断一个对象是否可以被回收](http://wuzhangyang.com/2019/01/12/how-do-jvm-kown-if-an-object-can-be-recycled/) ，但并不是一个对象不可达时，就宣告“死刑”的，此时只是暂时处于”缓刑“阶段，要宣告一个对象“死刑”，至少还要经历两次标记过程。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/java_object_self_redemption.png)



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

