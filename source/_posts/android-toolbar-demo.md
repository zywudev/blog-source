---
title: Android Toolbar 
date: 2018-05-01 20:16:04
tags:
---

记录一下 Toolbar 的基本使用：

1、确保 v7 support 库已添加。

```java
    implementation 'com.android.support:appcompat-v7:27.1.1'
```

2、Activity 继承自 AppCompatActivity。

3、修改配置文件 styles.xml 文件中默认主题，改为 NoActionBar，表示隐藏系统 ActionBar。

```java
 <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```

4、向 Activity 中添加 Toolbar 。

```java
<android.support.v7.widget.Toolbar
   android:id="@+id/toolbar"
   android:layout_width="match_parent"
   android:layout_height="?attr/actionBarSize"
   android:background="?attr/colorPrimary"
   android:elevation="4dp"
   android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
   app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
```

其中 elevation 4dp 是 Material Design 的建议仰角。

5、在 Activity 的 onCreate 方法中调用 setSupportActionBar 方法，设置 ToolBar 为应用栏。

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
    }
}
```

6、在 ToolBar 上面创建 Action 按钮：

首先创建 menu 文件夹（res---new---Directory)，然后创建 menu 文件。

```java
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/backup"
        android:icon="@drawable/ic_backup"
        android:title="Backup"
        app:showAsAction="always"/>
    <item
        android:id="@+id/delete"
        android:icon="@drawable/ic_delete"
        android:title="Delete"
        app:showAsAction="ifRoom"/>
    <item
        android:id="@+id/settings"
        android:icon="@drawable/ic_settings"
        android:title="Settings"
        app:showAsAction="never"/>
</menu>
```

其中，showAsAction 指定按钮的位置：

- always：永远显示在 ToolBar 中，空间不够不显示；
- ifRoom：屏幕空间足够的话显示在 ToolBar 中，不够的话显示在菜单中；
- never：永远显示在菜单中。

7、在 Activity 中，重写 两个方法：

```java
 @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.toolbar, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.backup:
                Toast.makeText(this,"click backup", Toast.LENGTH_SHORT).show();
                break;
            case R.id.delete:
                Toast.makeText(this,"click delete", Toast.LENGTH_SHORT).show();
                break;
            case R.id.settings:
                Toast.makeText(this,"click settings", Toast.LENGTH_SHORT).show();
                break;
        }
        return true;
    }
```

效果图：

![](http://om9o63aks.bkt.clouddn.com/275413748903691897.png)

[GitHub](https://github.com/zywudev/MDDemo)  