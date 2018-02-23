---
title: 设计模式之单例模式
date: 2017-04-24 09:09:54
---

## 什么是单例模式？

单例模式是一种对象创建型模式。所谓创建型模式就是将对象的创建和使用分离，在使用对象时无需关心对象的创建细节，从而降低系统的耦合度，使得设计方案更易于修改和扩展。



单例模式三个要点：（1）某个类只能有一个实例。（2）必须自行创建这个实例。（3）必须自行向整个系统提供这个实例。

## 饿汉式单例类

类加载进来就直接实例化对象，无需考虑多线程安全问题，但是浪费资源严重。

```java
public class EagerSingleton {
	// 类加载进入内存就创建单一的 instance 对象
	private static final EagerSingleton instance = new EagerSingleton();
	// 构造函数私有化，禁止外部类直接使用 new 来创建对象
	private EagerSingleton() {}
	// 提供一个全局的静态方法
	public static EagerSingleton getInstance() {
		return instance;
	}
}
```

## 懒汉式单例类

在第一次调用 `getInstance()` 方法时实例化，类加载时不自行实例化，在需要的时候再加载实例。但在多线程下会出现线程安全问题，可能会创建多个 instance 对象，违背了单例模式的初衷。

```java
public class LazySingleton {
	
	private static LazySingleton instance = null;
	
	private LazySingleton() {}
	
	public static LazySingleton getInstance() {
		if(instance == null) {
			instance = new LazySingleton();
		}
		return instance;
	}
}
```

为解决多线程问题，可以采用对 `getInstance()` 方法进行同步，但每次调用`getInstance()`方法都需要进行线程锁定判断，比较浪费资源，尤其在高并发访问环境下会导致系统性能大大降低。

```java
public class LazySingleton {
	
	private static LazySingleton instance = null;
	
	private LazySingleton() {}
	// 锁定 getInstance 方法
	public static synchronized LazySingleton getInstance() {
		if(instance == null) {
			instance = new LazySingleton();
		}
		return instance;
	}
}
```

事实上，无需对整个 `getInstance()` 方法锁定，只需要锁定代码 “ instance = new LazyInstance() ”。这种方法解决了浪费资源问题，但是在多线程下依然可能出现实例对象不唯一。原因在于：假如某一瞬间线程 A 和线程 B 都在调用`getInstance()` 方法，此时 instance 对象为 null，均能通过 “ instance == null ” 的判断。线程 A 进入 synchronized 锁定的代码中执行实例创建代码，线程 B 处于排队等待状态。当 A 执行完毕创建了实例后，线程 B 进入 synchronized 代码，此时 B 并不知道实例已经创建，将创建新的实例。

```java
public class LazySingleton {
	
	private static LazySingleton instance = null;
	
	private LazySingleton() {}
	
	public static synchronized LazySingleton getInstance() {
		if(instance == null) {
          	syncronized(LazySingleton.class) {    
            	instance = new LazySingleton();  
          	}	
		}
		return instance;
	}
}
```

因此还得进行改进，在 synchronized 锁定代码中再进行一次 “ instance == null ” 判断，这种方式称为**双重检查锁定** 。需要注意的是在需要在静态成员变量前加 `volatile` 修饰符。但 `volatile` 关键字会屏蔽 Java 虚拟机做的一些代码优化，导致系统运行效率降低。

```java
public class LazySingleton {
	
	private volatile static LazySingleton instance = null;
	
	private LazySingleton() {}
	
	public static synchronized LazySingleton getInstance() {
       // 第一重判断
		if(instance == null) {
          // 锁定代码块
          	syncronized(LazySingleton.class) {
                // 第二重判断
              	if(instance == null) {
                	instance = new LazySingleton();    
              	}
          	}	
		}
		return instance;
	}
}
```

## IoDH

饿汉式不能实现延迟加载，不管用不用，它始终占据内存；懒汉式 线程控制麻烦，而且性能受到影响。一种被称为 `Initialization on Demand Holder（IoDH）`的方法能够克服这些缺点。

静态单例对象没有作为 Singleton 的成员变量直接实例化，因此类加载时不会实例化 Singleton，第一次调用 `getInstance()` 时加载内部类 `HolderClass`，初始化 `instance`，由 Java 虚拟机保证其线程安全性，确保其只能初始化一次。

通过使用 IoDH，既可以实现延迟加载，又可以保证线程安全，不影响系统性能。

```java
public class Singleton {
	
	private Singleton() {}
	
	private static class HolderClass {
		private final static Singleton instance = new Singleton();
	}
	
	public static Singleton getInstance() {
		return HolderClass.instance;
	}
}
```

## 参考资料

[<设计模式的艺术之道>  刘伟](https://www.amazon.cn/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E7%9A%84%E8%89%BA%E6%9C%AF-%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91%E4%BA%BA%E5%91%98%E5%86%85%E5%8A%9F%E4%BF%AE%E7%82%BC%E4%B9%8B%E9%81%93-%E5%88%98%E4%BC%9F/dp/B00ATKMX9M/ref=sr_1_4?ie=UTF8&qid=1357551589&sr=8-4)

（完）





