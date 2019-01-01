---
title: Android 8.0 应用保活实践 
date: 2018-11-27 22:17:17
tags:
---

虽然我也觉得强行保活应用挺不厚道的，但是没办法，为了完成需求。

一开始尝试的方案是 Android 5.0 后系统提供的 JobScheduler，能够预先设置条件，达到条件时自动启动 JobService，在 Android 8.0 以下都能很愉快的使用。但是在华为的 Android 8.0 上，当应用被杀后，JobService 就不能被系统调用了。

于是采取了双进程服务绑定方式，实现了应用保活功能。

直接看原理图：

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/aidl-keep-alive.png)

原理就是利用 Binder 的讣告机制，如果 Service Binder 实体的进程被杀，系统会向 Client 发送讣告，这个时机就是保活的空子了。所以可以通过两个进程启动两个 Binder 服务，互为 C/S，一旦一个进程挂掉，另一个进程就会收到 Binder 讣告，这时可以拉起另一个进程。

那么图中两个进程中的 TransferActivity 是干什么用的 ，这个在后面再说。

这里我写了两个应用，一个是 AIDLServer，相当于服务端；一个是 AIDLClient，相当于客户端。而两个进程之间的通信采用 AIDL 方式。

```java
// IMyAidlInterface.aidl
package com.wuzy.aidlserver;

// Declare any non-default types here with import statements

interface IMyAidlInterface {

     void bindSuccess();

     void unbind();

}
```

注意两个应用的 AIDL 文件必须一致，包括包名。

然后，编写两个 binder 实体服务 RemoteService 、LocalService，主要代码如下：

```java
public class RemoteService extends Service {

    private static final String TAG = "RemoteService";
    
    @Override
    public void onCreate() {
        super.onCreate();
        Log.e(TAG, "onCreate: 创建 RemoteService");
        bindLocalService();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return stub;
    }
    
    private IMyAidlInterface.Stub stub = new IMyAidlInterface.Stub() {
        @Override
        public void bindSuccess() throws RemoteException {
            Log.e(TAG, "bindSuccess: LocalService 绑定 RemoteService 成功");
        }

        @Override
        public void unbind() throws RemoteException {
            Log.e(TAG, "unbind: 此处解除 RemoteService 与 LocalService 的绑定");
            getApplicationContext().unbindService(connection);
        }
    };

    /**
     * 绑定 LocalService
     */
    private void bindLocalService() {
        Intent intent = new Intent();
        intent.setComponent(new ComponentName("com.wuzy.aidlclient", "com.wuzy.aidlclient.LocalService"));
        if (!getApplicationContext().bindService(intent, connection, Context.BIND_AUTO_CREATE)) {
            Log.e(TAG, "bindLocalService: 绑定 LocalService 失败");
            stopSelf();
        }
    }

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
        
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            // bindRemoteService();
            createTransferActivity();
        }
    };

    private void createTransferActivity() {
        Intent intent = new Intent(this, TransferActivity.class);
        intent.setAction(TransferActivity.ACTION_FROM_SELF);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);
    }
}

```

```java
public class LocalService extends Service {

    private static final String TAG = "LocalService";
    
    @Override
    public void onCreate() {
        super.onCreate();
        Log.e(TAG, "onCreate: 创建 LocalService");
        bindRemoteService();

    }

    @Override
    public IBinder onBind(Intent intent) {
        Log.e(TAG, "onBind: 绑定 LocalService");
        return stub;
    }

    private IMyAidlInterface.Stub stub = new IMyAidlInterface.Stub() {
        @Override
        public void bindSuccess() throws RemoteException {
            Log.e(TAG, "bindSuccess: RemoteService 绑定 LocalService 成功");
        }

        @Override
        public void unbind() throws RemoteException {
            getApplicationContext().unbindService(connection);
        }
    };

    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
       
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            // bindRemoteService();
            createTransferActivity();
        }
    };

    private void createTransferActivity() {
        Intent intent = new Intent(this, TransferActivity.class);
        intent.setAction(TransferActivity.ACTION_FROM_SELF);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);
    }

    private void bindRemoteService() {
        Intent intent = new Intent();
        intent.setComponent(new ComponentName("com.wuzy.aidlserver", "com.wuzy.aidlserver.RemoteService"));
        if (!getApplicationContext().bindService(intent, connection, Context.BIND_AUTO_CREATE)) {
            Log.e(TAG, "bindRemoteService: 绑定 RemoteService 失败");
            stopSelf();
        }
    }
}

```

在 onCreate 的时候相互绑定，并在 onServiceDisconnected 收到讣告的时候，重新启动服务绑定彼此即可。

但是我在系统是 8.0 的华为机器上是无效的，也就是当 LocalService 所在进程被杀后，RemoteService 无法启动LocalService，反过来也是如此。

所以，这里只能采取 "曲线救国" 的方式。通过 TransferActivity 中转下，先启动守护进程的 TransferActivity，再从守护进程的 TransferActivity 中启动保活进程的 TransferActivity，这是没有问题的，再从保活进程的 TransferActivity 中启动 LocalService，重新绑定服务即可，反过来也是一样的。当然，TransferActivity 要用户无感知，不然会很突兀，所以这里的 TransferActivity 都是 1 个像素，做完任务及时销毁即可。

TransferActivity 的代码就不贴了，具体可以去 [GitHub](https://github.com/zywudev/AndroidKeepAlive) 了解。

这种方式用来保活一般是没有问题的，因为 Binder 讣告是系统中 Binder 框架自带的，除非一次性杀了两个进程，那就没辙了。

最后，一般保活的目的是为了做某项任务，所以，任务完成后应该结束保活功能，不然老是占着内存确实挺不厚道的。

