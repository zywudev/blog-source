---
title: Java Exception and Error 
date: 2018-06-01 21:03:49
tags: Java
toc: true
---

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/Java%E5%BC%82%E5%B8%B8.png)

1、Exception 和 Error 都是继承自 Throwable 类。

2、Exception 包括可检查异常和不检查异常。可检查异常需要在代码中进行显式捕获；不检查异常又叫运行时异常，不强求在代码中捕获。

3、Error 是指在正常情况下，不太会出现的错误，绝大部分 Error 都会导致程序处于非正常、不可恢复状态。

4、尽量不要捕获 Throwable 或者 Error，这样很难保证我们能正确处理。

5、不要生吞异常。对于不知道怎么处理的异常可以直接抛出去或者构建新的异常抛出去，在更高层面有了清晰的业务，往往更清楚合适的处理方式。

6、try-catch 会产生额外的性能开销，所以建议仅仅捕获有必要的代码段。

7、自定义异常：对特有的问题进行自定义异常封装，捕获处理异常。可见继承自 Exception 或者 Throwable，不要继承自 Error。

8、try-with-resources: 一个声明一个或多个资源的 try 语句。一个资源作为一个对象，必须在程序结束之后随之关闭。 try-with-resources 语句确保在语句的最后每个资源都被关闭 。任何实现了 java.lang.AutoCloseable 的对象, 包括所有实现了 java.io.Closeable 的对象, 都可以用作一个资源。

```java
public static void readAll() {
    try (
        FileReader fileReader = new FileReader("test");
        BufferedReader bufferedReader = new BufferedReader(fileReader)
    ) {
        String line;

        while ((line = bufferedReader.readLine()) != null) {
            System.out.println(line);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

一旦 `bufferedReader.readLine()` 出现异常，fileReader 和 bufferedReader 会自动关闭。

9、multiple-catch : 将多个异常合并写法，简化代码。

```java
catch (IOException|SQLException ex) {
    logger.log(ex);
    throw ex;
}
```

注意 `|` 两边的异常不能相交，即不能有父子继承关系。以上两种特性均是 Java7 之后的特性。





