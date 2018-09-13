---
title: Java 反射基础
date: 2018-09-13 22:08:02
tags:
toc: true
---

最近在调研 Android 应用加固方案，涉及大量反射技术，因此趁这个机会总结下 Java 反射的一些知识。

## 什么是反射？

反射是 Java 语言提供的一种基本功能。通过反射我们可以在运行时动态地操作类或者对象，比如获取某个对象的类定义，获取类声明的属性和方法，调用方法或者构造函数，甚至可以在运行时修改类定义。

## 基本使用方法

反射的主要步骤包括：

- 获取目标类型的 Class 对象
- 通过 Class 对象分别获取 Constructor 类对象、Method 类对象 和 Field 类对象。
- 通过 Constructor 、Method 和 Field 分别获取目标类的构造函数、方法和属性的具体信息，并进行后续操作。

### 获取目标类型的 Class 对象

**1、Object.getClass()**

```java
StringBuilder stringBuilder = new StringBuilder("123");
Class<?> classType = stringBuilder.getClass();
System.out.println(classType);   // class java.lang.StringBuilder
```

**2、T.class** 

```java
Class<?> classType = int.class;
System.out.println(classType);   // int
```

T 代表任意 Java 类型。

**3、Class.forName()**

```java
Class<?> classType = null;
try {
    classType = Class.forName("java.lang.Integer");
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
System.out.println(classType);   // class java.lang.Integer
```

### 通过 Class 对象分别获取 Constructor 类对象、Method 类对象 和 Field 类对象

**1、获取 Constructor 类对象**

```java
// a. 获取指定的构造函数 （公共 / 继承）
Constructor<T> getConstructor(Class<?>... parameterTypes)
// b. 获取所有的构造函数（公共 / 继承） 
Constructor<?>[] getConstructors(); 
// c. 获取指定的构造函数 （不包括继承）
Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) 
// d. 获取所有的构造函数（不包括继承）
Constructor<?>[] getDeclaredConstructors(); 
```

**2、获取 Method 类对象**

```java
// a. 获取指定的方法（公共 / 继承）
Method getMethod(String name, Class<?>... parameterTypes) ；
// b. 获取所有的方法（公共 / 继承）
Method[] getMethods() ；
// c. 获取指定的方法 （ 不包括继承）
Method getDeclaredMethod(String name, Class<?>... parameterTypes) ；
// d. 获取所有的方法（ 不包括继承）
Method[] getDeclaredMethods() ；
```

**3、获取 Field 类对象**

```java
// a. 获取指定的属性（公共 / 继承）
Field getField(String name) ;
// b. 获取所有的属性（公共 / 继承）
Field[] getFields() ;
// c. 获取指定的所有属性 （不包括继承）
Field getDeclaredField(String name) ；
// d. 获取所有的所有属性 （不包括继承）
Field[] getDeclaredFields() ；
```

以上方法中，不带 "Declared" 的方法返回某个类的公共方法或属性，继承的方法或属性；带 "Declared" 的方法返回公共、保护、默认（包）访问和私有方法或属性，但不包括继承的方法或属性。

### 通过 Constructor 、Method 和 Field 分别获取目标类的构造函数、方法和属性的具体信息，并进行后续操作

**1、利用反射调用类的构造函数**

```java
// 获取 ConstructorClass 类的 Class 对象
Class clazz = ConstructorClass.class;
Object obj1 = clazz.getConstructor().newInstance(); // 输出：无参数构造函数
Object obj2 = clazz.getConstructor(String.class).newInstance("wuzy"); // 输出：有参数构造函数

// 目标类
class ConstructorClass {

    public ConstructorClass() {
        System.out.println("无参数构造函数");
    }

    public ConstructorClass(String str) {
        System.out.println("有参数构造函数");
    }
}

```

newInstance() 调用默认构造函数，若目标类无构造函数，则抛出异常 NoSuchMethodException。

**2、利用反射调用类对象的方法**

```java
// 1、获取 MethodClass 类的 Class 对象
Class<?> clazz = MethodClass.class;
// 2、通过 Class 创建 MethodClass 对象
Object object = clazz.newInstance();
// 3、通过 Class 对象获取 add 方法
Method method = clazz.getMethod("add", int.class, int.class);
// 4、通过 Method 调用 add 方法
Object result = method.invoke(object,1,4);
System.out.println(result);   // 输出 5

// 目标类
class MethodClass {

    public MethodClass() {
    }

    public int add(int a, int b) {
        return a + b;
    }
}
```

**3、利用反射获取类的属性**

```java
Class clazz = FieldClass.class;
Object object = clazz.newInstance();
Field field = clazz.getDeclaredField("name");
field.setAccessible(true);
field.set(object, "wuzy");
System.out.println(field.get(object));   // 输出： wuzy

// 目标类
class FieldClass {

    public FieldClass() {}

    private String name;
}
```

其中， setAccessible 用于屏蔽 Java 语言的访问检查，设为 true 可以访问类的私有属性、方法。

## 反射的使用场景

1、工厂模式：Factory 类中用反射的话，添加了一个新的类之后，就不需要再修改工厂类 Factory 了

2、数据库 JDBC 中通过 Class.forName(Driver) 来获得数据库连接驱动

3、访问一些不能访问的变量或属性：破解别人代码。

4、实现动态代理。

以上就是反射的基本知识点，需要注意的是由于反射会额外消耗一定的系统资源，因此如果不需要动态地创建一个对象，那么就不需要用反射。通过反射实现动态代理和工厂模式会在后续文章中专门撰写。