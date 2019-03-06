---
title: OKHttp 源码分析（一）—— 基本流程
date: 2019-03-04 09:37:43
tags:
---

## 基本使用

```java
 // 1、创建 OKHttpClient
OkHttpClient client = new OkHttpClient();

// 2、创建 Request
Request request = new Request.Builder()
    .get()
    .url("xxx")
    .build();

// 3、创建 Call
Call call = client.newCall(request);

try {
    // 4、同步请求
    Response response = call.execute();
} catch (IOException e) {
    e.printStackTrace();
}

// 5、异步请求
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {

    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {

    }
});
```

### 1、创建 OKHttpClient

OKHttpClient 支持两种构造方式。

默认方式：不需要配置任何参数，基本参数都是默认的。

```java
 public OkHttpClient() {
     this(new Builder());
 }
```

调用的是下面的这个构造函数：

```java
OkHttpClient(Builder builder) {...}
```

建造者模式：通过 Builder 配置参数，最后通过 `build` 方法返回一个 OkHttpClient 实例。

```java
public OkHttpClient build() {
    return new OkHttpClient(this); // 这里的 this 是 Builder 实例
}
```

### 2、创建 Request

Request 未对外提供 public 的构造函数，所以构建一个 Request 实例需要使用构造者模式构建。

```java
Request(Builder builder) {
    this.url = builder.url;
    this.method = builder.method;
    this.headers = builder.headers.build();
    this.body = builder.body;
    this.tags = Util.immutableMap(builder.tags);
}
```

### 3、创建 Call

`OkHttpClient` 实现了 `Call.Factory`，负责根据 Request 创建新的 Call，我们来看具体代码：

```java
@Override
public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
}
```

这里我们发现实际上调用了RealCall 的静态方法 `newRealCall`， 不难猜测 这个方法就是创建 Call 实例。

```java
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    call.eventListener = client.eventListenerFactory().create(call);
    return call;
  }
```

### 4、同步请求

同步请求调用的是 RealCall 的 `execute` 方法。

```java
@Override public Response execute() throws IOException {
    synchronized (this) {
      // （1）每个 call 只能执行一次
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    captureCallStackTrace();
    timeout.enter();
    eventListener.callStart(this);
    try {
      // （2）通知 dispatcher 已经进入执行状态
      client.dispatcher().executed(this);
      // （3）通过一系列拦截器请求处理和响应处理得到最终的返回结果
      Response result = getResponseWithInterceptorChain();
      if (result == null) throw new IOException("Canceled");
      return result;
    } catch (IOException e) {
      e = timeoutExit(e);
      eventListener.callFailed(this, e);
      throw e;
    } finally {
      //（4）通知 dispatcher 自己已经执行完毕
      client.dispatcher().finished(this);
    }
  }
```

这里主要做了这几件事：

- 检测这个 call 是否已经执行了，保证每个 call 只能执行一次。
- 通知 dispatcher 已经进入执行状态，将请求加入到同步队列中。
- 调用 `getResponseWithInterceptorChain()` 函数获取 HTTP 返回结果。
- 最后还要通知 `dispatcher` 自己已经执行完毕

### 5、异步请求



