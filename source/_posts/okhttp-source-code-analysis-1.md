---
title: OkHttp 源码分析（一）—— 请求流程
date: 2019-03-04 09:37:43
tags:
---

OkHttp 是目前最为流行的网络请求框架，了解 OkHttp 的实现原理和设计思想是很有必要的。

这篇文章主要梳理一下 OkHttp 的请求流程，对 OkHttp 的实现原理有个整体的把握，再深入细节的实现会更加容易。

建议将 OkHttp 的源码下载下来，使用 IDEA 编辑器可以直接打开阅读。我这边也将最新版的源码下载下来，进行了注释，有需要的可以直接从  [这里](https://github.com/zywudev/android_source_code_analysis) 下载查看。

一图胜千言，我们先看一下 OkHttp 的请求流程图。



```java
// 1、创建 Request
Request request = new Request.Builder()
    .get()
    .url("xxx")
    .build(); 

// 2、创建 OKHttpClient
OkHttpClient client = new OkHttpClient();

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





## Request

Request 类封装了一次请求需要传递给服务端的参数：请求 method 如 GET/POST 等、一些 header、RequestBody 等等。

Request 类未对外提供 public 的构造函数，所以构建一个 Request 实例需要使用构造者模式构建。

```java
Request(Builder builder) {
    this.url = builder.url;
    this.method = builder.method;
    this.headers = builder.headers.build();
    this.body = builder.body;
    this.tags = Util.immutableMap(builder.tags);
}
```

## OkHttpClient

OkHttpClient 支持两种构造方式。

默认方式：

```java
OkHttpClient client = new OkHttpClient();
```

看一下构造函数：

```java
public OkHttpClient() {
     this(new Builder());
 }
```



```java
OkHttpClient(Builder builder) {...}
```

建造者模式：通过 Builder 配置参数，最后通过 `build` 方法返回一个 OkHttpClient 对象。

```java
OkHttpClient client = new OkHttpClient.Builder().build();

public OkHttpClient build() {
    return new OkHttpClient(this); // 这里的 this 是 Builder 实例
}
```

## Call

`OkHttpClient` 实现了 `Call.Factory`，负责根据 Request 创建新的 Call：

```java
Call call = client.newCall(request);
```

看一下 `newCall` 方法。

```java
@Override
public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
}
```

这里我们发现实际上调用了 RealCall 的静态方法 `newRealCall`， 不难猜测 这个方法就是创建 Call 实对象。

```java
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    call.eventListener = client.eventListenerFactory().create(call);
    return call;
  }
```

## 同步请求

从上面的分析我们知道，同步请求调用的实际是 RealCall 的 `execute` 方法。

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
      // （2）请求开始, 将自己加入到runningSyncCalls队列中
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
      //（4）请求完成, 将其从runningSyncCalls队列中移除
      client.dispatcher().finished(this);
    }
  }
```

这里主要做了这几件事：

- 检测这个 call 是否已经执行了，保证每个 call 只能执行一次。
- 通知 dispatcher 已经进入执行状态，将 Call 加入到 runningSyncCalls 队列中。
- 调用 `getResponseWithInterceptorChain()` 函数获取 HTTP 返回结果。
- 最后还要通知 `dispatcher` 自己已经执行完毕，将 Call 从 runningSyncCalls 队列中移除

可以看到真正发出网络请求，解析返回结果的是在 `getResponseWithInterceptorChain` 方法中进行的。

```java
Response getResponseWithInterceptorChain() throws IOException {
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }

    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());
    
    return chain.proceed(originalRequest);
```

从 `getResponseWithInterceptorChain` 方法我们可以看出 Interceptor（拦截器）是 OkHttp 的精髓。

这里先是创建了一个 Interceptor 的集合，然后将各类 interceptor 全部加入到集合中，包含以下 interceptor：

- interceptors：配置 OkHttpClient 时设置的 inteceptors

- RetryAndFollowUpInterceptor：负责失败重试以及重定向
- BridgeInterceptor：负责把用户构造的请求转换为发送到服务器的请求、把服务器返回的响应转换为用户友好的响应
- CacheInterceptor：负责读取缓存直接返回、更新缓存
- ConnectInterceptor：负责和服务器建立连接
- networkInterceptors：配置 OkHttpClient 时设置的 networkInterceptors
- CallServerInterceptor：负责向服务器发送请求数据、从服务器读取响应数据

添加完拦截器后，创建了一个 RealInterceptorChain 对象，将集合 interceptors 和 index（**数值0**）传入。接着调用其 `proceed` 方法进行请求的处理，我们来看 `proceed`方法。

```java
public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      RealConnection connection) throws IOException {
    if (index >= interceptors.size()) throw new AssertionError();
    ...
    // 创建下一个RealInterceptorChain，将index+1（下一个拦截器索引）传入
    RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
        connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
        writeTimeout);
    // 获取当前的拦截器
    Interceptor interceptor = interceptors.get(index);
    // 通过Interceptor的intercept方法进行处理
    Response response = interceptor.intercept(next);
    ...
    return response;
  }
```

我们来看一些关键代码：

RealInterceptorChain 的 `proceed` 方法先创建 RealInterceptorChain 的对象，将集合 interceptors 和 index + 1 传入。从前面的分析指导，初始 index 为 0。

然后获取当前 index 位置上的 Interceptor，将创建的 RealInterceptorChain 对象 next 传入到当前拦截器的 `intercept` 方法中，`intercept` 方法内部会调用 next 的 proceed 方法，一直递归下去，最终完成一次网络请求。

所以每个 Interceptor 主要做两件事情：

- 拦截上一层拦截器封装好的 Request，然后自身对这个 Request 进行处理，处理后向下传递。
- 接收下一层拦截器传递回来的 Response，然后自身对 Response 进行处理，返回给上一层。

## 异步请求

异步请求调用的是 RealCall 的 `enqueue` 方法。

```java
 public void enqueue(Callback responseCallback) {
     synchronized(this) {
         if (this.executed) {
             throw new IllegalStateException("Already Executed");
         }

         this.executed = true;
     }

     this.captureCallStackTrace();
     this.eventListener.callStart(this);
     this.client.dispatcher().enqueue(new RealCall.AsyncCall(responseCallback));
 }
```

这里有个重要的的参与者  Dispatcher，它的作用是：控制每一个 Call 的执行顺序和生命周期。

- 对于同步请求，由于它是即时运行的， Dispatcher 只需要运行前请求前存储到同步队列中，请求结束后从同步队列中移除即可。
- 对于异步请求，Dispatcher 首先会启动 ExecutorService，不断取出异步请求去进行执行，最多执行的异步请求数量为64，如果已经有 64 个 Request 在执行，那么将这个 Request 存在等待队列中。

我们看一下他的 `enqueue` 方法。

```
void enqueue(AsyncCall call) {
  // 将AsyncCall加入到准备异步调用的队列中
  synchronized (this) {
    readyAsyncCalls.add(call);
  }
  promoteAndExecute();
}
```

继续看 `promoteAndExecute` 方法。

```java
private boolean promoteAndExecute() {
    assert (!Thread.holdsLock(this));

    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;
    synchronized (this) {
        for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
            AsyncCall asyncCall = i.next();

            if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
            if (runningCallsForHost(asyncCall) >= maxRequestsPerHost) continue; // Host max capacity.

            i.remove();
            executableCalls.add(asyncCall);
            runningAsyncCalls.add(asyncCall);
        }
        isRunning = runningCallsCount() > 0;
    }

    for (int i = 0, size = executableCalls.size(); i < size; i++) {
        AsyncCall asyncCall = executableCalls.get(i);
        asyncCall.executeOn(executorService());
    }

    return isRunning;
}
```

这里主要的工作有：

- 从准备异步请求的队列中取出可以执行的请求（当前请求数小于最大请求容量；同一个主机请求数小于最大请求数量），加入到 `executableCalls` 列表中。
- 循环 `executableCalls` 取出请求 AsyncCall 对象，调用其 `executeOn` 方法。 

```java
void executeOn(ExecutorService executorService) {
    assert (!Thread.holdsLock(client.dispatcher()));
    boolean success = false;
    try {
        executorService.execute(this);
        success = true;
    } catch (RejectedExecutionException e) {
        InterruptedIOException ioException = new InterruptedIOException("executor rejected");
        ioException.initCause(e);
        eventListener.callFailed(RealCall.this, ioException);
        responseCallback.onFailure(RealCall.this, ioException);
    } finally {
        if (!success) {
            client.dispatcher().finished(this); // This call is no longer running!
        }
    }
}
```

可以看到 `executeOn` 方法的参数传递的是 ExecutorService 线程池对象，所以 AsyncCall 应该是实现了 Runnable 接口，我们看看它的 `run` 方法是怎样的。

AsyncCall 继承自 NamedRunnable 抽象类。

```java
public abstract class NamedRunnable implements Runnable {
  protected final String name;

  public NamedRunnable(String format, Object... args) {
    this.name = Util.format(format, args);
  }

  @Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
}
```

所以当线程池执行 execute 方法会走到 NamedRunnable 的 run 方法，run 方法中又调用了 抽象方法 execute，我们直接看 AsyncCall 的 execute 方法。

```java
@Override
protected void execute() {
    boolean signalledCallback = false;
    timeout.enter();
    try {
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
            signalledCallback = true;
            responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        } else {
            signalledCallback = true;
            responseCallback.onResponse(RealCall.this, response);
        }
    } catch (IOException e) {
        e = timeoutExit(e);
        if (signalledCallback) {
            // Do not signal the callback twice!
            Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
            eventListener.callFailed(RealCall.this, e);
            responseCallback.onFailure(RealCall.this, e);
        }
    } finally {
        client.dispatcher().finished(this);
    }
}
}
```

这里我们又看到了熟悉的 `getResponseWithInterceptorChain` 方法。

## 总结







 