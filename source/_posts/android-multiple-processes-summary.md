---
title: Android 多进程基本知识整理
date: 2017-09-08 14:06:00
tags:
---

在 Android 中，一个应用默认有一个进程。但我们可以通过配置实现在一个应用中开启多个进程。

### 开启多进程方式

- 在清单文件中指定 `android:process` 属性
- 适用元素：Activity，Service，Receiver，ContentProvider。
- 以`:` 开头，表示这个进程是应用的私有进程；以小写字母开头，表示这个进程是全局进程，可以被多个应用公用。

```xml
<activity
    android:name=".SecondActivity"
    android:process=":second" />
<activity
    android:name=".ThirdActivity"
    android:process="com.example.wuzy.helloworld.third"/>
```

### 多进程的优点

系统为每个应用分配一定大小的内存，从之前的 16M 到 32M、48M，甚至更高。但毕竟有限。

进程是资源分配的基本单位。也就是说，一个应用有对个进程，那这个应用可以获得更多的内存。

所以，**开启多进程可以分担主进程的内存消耗**，常见音乐类 APP 的后台播放，应用的推送服务等。

### 多进程的不足

**1、数据共享问题**

Android 系统为每个进程分配独立的虚拟机，不同的虚拟机之间数据不能共享，即使是静态成员还是单例模式。

**2、线程同步机制失效**

不同进程锁的不是同一个对象，无法保证线程同步了。

**3、SharedPreferences 可靠性下降**

SharedPreferences 还没有增加对多进程的支持。

**4、Application 多次创建**

当一个组件跑在新的进程中，系统要在创建进程的同时为其分配独立的虚拟机，自然就会创建新的 Application。这就导致了 application 的 onCreate方法重复执行全部的初始化代码。因此，可以根据进程需要进行最小的业务初始化。

**不同进程共同的初始化业务逻辑** ：

```java
public class AppInitialization {

    /**
     * 不同进程共同的初始化代码
     * @param application
     */
    public void onAppCreate(Application application) {
        // init
    }
```

**简单工厂模式** ：

根据进程名进行对应进程的初始化逻辑。

```java
public class AppInitFactory {
 
    public static AppInitialization getAppInitialization(String processName) {
        AppInitialization appInitialization;
        if (processName.endsWith(":second")) {
            appInitialization = new SecondApplication();
        } else if (processName.endsWith(":third")) {
            appInitialization = new ThirdApplication();
        } else {
            appInitialization = new AppInitialization();
        }
        return appInitialization;
    }

    static class SecondApplication extends AppInitialization {

        @Override
        public void onAppCreate(Application application) {
            super.onAppCreate(application);
            // init
        }
    }

    static class ThirdApplication extends AppInitialization {
        @Override
        public void onAppCreate(Application application) {
            super.onAppCreate(application);
            // init
        }
    }
```

**具体调用代码** ：

```java
public class MyApplication extends Application {

    private static final String TAG = "MyApplication";

    @Override
    public void onCreate() {
        super.onCreate();
        String currentProcessName = getCurrentProcessName();
        Log.e(TAG, "currentProcessName : " + currentProcessName );
        AppInitialization appInitialization =                                AppInitFactory.getAppInitialization(currentProcessName);
        if (appInitialization != null) {
            appInitialization.onAppCreate(this);
        }
    }

    /**
     * 获取当前进程名称
     * @return
     */
    private String getCurrentProcessName() {
        String currentProcessName = "";
        int pid = android.os.Process.myPid();
        ActivityManager manager = (ActivityManager) this.getSystemService(Context.ACTIVITY_SERVICE);
        for (ActivityManager.RunningAppProcessInfo processInfo : manager.getRunningAppProcesses()) {
            if (processInfo.pid == pid) {
                currentProcessName = processInfo.processName;
                break;
            }
        }
        return currentProcessName;
    }
}
```


