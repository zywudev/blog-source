---
title: 设计模式之工厂方法模式
date: 2017-06-15 19:38:00
---

简单工厂模式提供了专门的工厂类用于创建对象，实现了对象的创建和使用分离。但工厂类集中了所有产品对象的创建逻辑，职责过重；此外，如果增加新产品，需要修改工厂类的源代码，违背开闭原则。

工厂方法模式则可以很好的解决这一问题。在工厂方法模式中，不在提供统一的工厂类创建所有的产品对象，而是针对不同的产品提高不同的工厂。

模式结构图如下：

![](http://om9o63aks.bkt.clouddn.com/factory-method-pattern.jpg)

共包含以下 4 个角色：

- Product：定义产品的接口。也就是产品对象的公共父类。
- ConcreteProduct：具体产品类，与具体工厂一一对应。
- Factory：抽象工厂接口。它是工厂方法模式的核心。
- ConcreteFactory：具体工厂类，返回一个具体产品类的实例。


**工厂模式代码：**

```java
// 抽象产品类
public interface Product {
	public void productMethod();
}

// 具体产品类
public class ConcreteProduct implements Product {

	@Override
	public void productMethod() {
		System.out.println("具体产品");
	}
}

// 抽象工厂类
public interface Factory {
	public Product factoryMethod(); 
}

// 具体工厂类
public class ConcreteFactory implements Factory { 
	@Override
	public Product factoryMethod() {
		// TODO Auto-generated method stub
		return new ConcreteProduct();
	}
}

// 客户端测试代码
public class Client {

	public static void main(String[] args) {
		Factory  factory;
		Product product;
		factory = new ConcreteFactory();
		product = factory.factoryMethod();
		product.productMethod();
	}
}
```

工厂方法模式主要优点：

- 用户只需要关心所需产品对应的工厂，无需关心创建细节，甚至无需知道具体产品类的类名。
- 在系统中加入新产品时，无需修改抽象工厂和抽象产品接口，无需修改客户端，也无需修改其他的具体工厂和具体产品，只需要加入一个具体工厂和具体产品就可以。系统的可扩展性非常好。

主要缺点：

- 添加新产品时，系统中类的个数成对增加，增加了系统的复杂性。同时，抽象层增加了系统的抽象性和理解难度。



