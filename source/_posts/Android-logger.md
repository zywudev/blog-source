---
title: Android 专用的日志封装库
date: 2018-07-01 11:51:59
tags: 
- Android 日志
- 开源库
categories: Android
---

俗话说，要想程序不出 Bug， 那就一行代码也不写。

所以在程序开发或者上线后如果出现了 Bug，能够及时查看日志，对修复 Bug 非常有帮助。

目前最为流行的本地日志框架应该是 orhanobut 的 [Logger](https://github.com/orhanobut/logger) 库，功能很强大而且打印出来的日志非常好看。网络日志这块应该是 square 的 [okhttp-logging-interceptor](https://github.com/square/okhttp/tree/master/okhttp-logging-interceptor) 库。

于是我便对这两种框架进行了封装，作为日常日志工具。这里推荐给大家使用。

### 支持以下功能

- Logcat 后台打印好看整洁的日志。
- 应用崩溃日志和 error 级别日志自动保存至本地文件。
- Logcat 后台打印 Http 日志，屏蔽了文件流打印乱码。

### 使用方法

1、引入依赖

```java
implementation 'com.wuzy:logger:1.0.0'
```

2、在 Application 中初始化：

```java
L.init(tag, isLoggable, packageName, appName);
```

其中 `tag`为日志标识，`isLoggable` 是否支持打印后台日志，`packageName` 为包名， `appName` 为应用名称。

应用崩溃日志和 error 级别日志会自动保存至内部存储路径 `Android/data/packageName/log/` 路径下。

3、打印不同级别日志：

```java
L.d("message1");

L.w("message2");

L.i("message3");

L.json("{ \"key\": 3, \"value\": something}");

Map<String, String> map = new HashMap<>();
map.put("key", "value");
map.put("key1", "value2");

L.d(map);

L.e(new Throwable("error"));
```

4、打印 OKHttp 网络日志：

```java
HttpLogInterceptor logger = new HttpLogInterceptor();
logger.setLevel(HttpLogInterceptor.Level.BODY);
OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .addInterceptor(logger)
    .build();
```

如果在使用的过程中出现问题，大家去 [GitHub](https://github.com/zywudev/Logger) 提 Issues，也可以自行修改。