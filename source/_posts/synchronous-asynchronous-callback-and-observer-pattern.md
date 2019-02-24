---
title: 同步、异步、回调及观察者模式
date: 2017-08-15 10:12:08
tags: 
- 设计模式
- Java
toc: true
---

### **1、同步调用**

同步调用是最基本的调用方式，即类 A 的方法 a() 调用类 B 的方法 b()，一直等待方法 b() 执行结束，方法 a() 才继续往下走。这是一种阻塞调用， 适用于方法 b() 执行时间不长的情况，如果方法 b() 执行时间过长或者直接阻塞的话，方法 a() 的余下代码是无法执行下去的。

### **2、异步调用**

异步调用是为了解决同步调用存在阻塞情况而产生的一种调用方式。类 A 的方法 a() 通过创建新线程的方式调用类 B 的方法 b()，代码接着直接往下执行。这样方法 b() 在新线程中执行，就不会阻塞方法 a() 的执行。对于方法 a() 无需方法 b() 的返回结果，可以直接往下执行其他操作。对于需要方法 b() 的返回结果，通常可以借助回调机制实现。

### **3、回调**

**定义**：

> 回调函数就是一个通过函数指针调用的函数。如果你把函数的指针（地址）作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，我们就说这是回调函数。回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。



函数指针对应于 Java 中的引用，使用更加简单。

Android 中的 Button 点击事件就是应用了回调机制，这其实就是回调最常见的应用场景之一。**我们自己不会显式地去调用 onClick 方法。用户触发了该按钮的点击事件后，它会由 Android 系统来自动调用**。

**原始代码**：

```java
Button button = (Button) findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
      Toast.makeText(MainActivity.this, "按钮被点击了。", Toast.LENGTH_SHORT).show();
    }
});
```

**模拟 Button 监听事件**：

**监听接口**：

```java
public interface MyOnClickListener {
    // 回调函数
	public void onClick();	
}
```

**Button 类**：

```java
public class MyButton {
	MyOnClickListener listener;	
	public void setOnClickListener(MyOnClickListener listener) {
		this.listener = listener;
	}
		
	/*
	 * 按钮被点击
	 */
    public void click() {
    	listener.onClick();
    }
}
```

**注册监听器和触发点击操作**：

```java
public class Client {
	
	public static void main(String[] args) {
		MyButton button = new MyButton();	
      
		// 注册监听器
		button.setOnClickListener(new MyOnClickListener() {
			
			@Override
			public void onClick() {
				System.out.println("按钮被点击了");	
			}
		});
		
		// 用户点击按钮
		button.click();
	}	
}
```

与原始 android 点击 Button 事件对比，可以发现原始 android 点击不需要调用 click 事件。这是为什么呢？

实际上，在源码中，当 android 系统检测到该 View 的 ACTION_UP 的操作时，会调用 performClick() 这个函数，而这个函数的内容如下：

```java
public boolean performClick() {  
	sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);  
    if (mOnClickListener != null) {  
        playSoundEffect(SoundEffectConstants.CLICK);  
        mOnClickListener.onClick(this);  
        return true;  
    }  
    return false;  
}  
```

只要 mOnClickListener 不为 null，那么 onClick 就会被调用。所以，它和模拟点击原理其实是一样的。

### **4、观察者模式**

**定义**：

> 观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态上发生变化时，会通知所有观察者对象，使它们能够自动更新自己。

**观察者接口**：

```java
public interface Observer {
	public void update();
}
```

**主题对象**：

```java
public class Subject {
	
	List<Observer> lists = new ArrayList<Observer>();
	
	public void register(Observer observer) {
		lists.add(observer);
	}
	
	public void notifyObservers() {
		for(Observer observer : lists) {
			observer.update();
		}
	}
	
	public void unRegister(Observer observer) {
		lists.remove(observer);
	}
}
```

**注册更新观察者**：

```java
public class Client {
	
	public static void main(String[] args) {
		
		Subject subject = new Subject();
		
		subject.register(new Observer() {
			
			@Override
			public void update() {
				System.out.println("onserver 1 update");
			}
		});
		
		subject.register(new Observer() {
			
			@Override
			public void update() {
				System.out.println("observer 2 update");
			}
		});
		
		subject.notifyObservers();
	}
}
```

**这样看来，回调机制和观察者模式是一致的，区别是观察者模式里面目标类维护了所有观察者的引用，而回调里面只是维护了一个引用。**



