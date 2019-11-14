---
title: android-interview-6-anr
date: 2019-11-14 15:36:55
tags: 
---

![](android-interview-6-anr/anr_dialog.png)



获取日志的命令：

```shell
adb shell
// 
cat /data/anr/traces.txt > /mnt/sdcard/traces.txt
//
exit
//
adb pull /sdcard/traces.txt
```





```shell
----- pid 13872 at 2019-11-14 15:18:50 -----
Cmd line: com.wuzy.anrtest

...

"main" prio=5 tid=1 Sleeping
  | group="main" sCount=1 dsCount=0 flags=1 obj=0x735f1ad0 self=0x7b396a3a00
  | sysTid=13872 nice=-10 cgrp=default sched=0/0 handle=0x7b3e6f69b0
  | state=S schedstat=( 832049486 21080201 432 ) utm=77 stm=6 core=5 HZ=100
  | stack=0x7fe0065000-0x7fe0067000 stackSize=8MB
  | held mutexes=
  at java.lang.Thread.sleep(Native method)
  - sleeping on <0x026e2fdc> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:386)
  - locked <0x026e2fdc> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:327)
  at com.wuzy.anrtest.MainActivity$1.onClick(MainActivity.java:19)
  at android.view.View.performClick(View.java:6291)
  at android.view.View$PerformClick.run(View.java:24931)
  at android.os.Handler.handleCallback(Handler.java:808)
  at android.os.Handler.dispatchMessage(Handler.java:101)
  at android.os.Looper.loop(Looper.java:166)
  at android.app.ActivityThread.main(ActivityThread.java:7523)
  at java.lang.reflect.Method.invoke(Native method)
  at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:245)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:921)
```

