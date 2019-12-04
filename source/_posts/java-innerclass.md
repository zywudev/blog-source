---
title: Java 内部类总结
date: 2019-02-27 10:31:37
tags: Java 基础
categories: Java
---

Java 中，可以将一个类定义在另一个类或者一个方法里面，这样的类称为内部类。

一般包含四种内部类：成员内部类、匿名内部类、局部内部类和静态内部类。

## 成员内部类

成员内部类的定义位于另一个类的内部，形式如下：

```java
public class Outer {

    private String name;

    class Inner {
        Inner(){
            name = "wuzy";
        }

        private void displayName() {
            System.out.println(name);
        }
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner();
        inner.displayName();  // wuzy
    }
}
```

成员内部类可以无条件访问外部类的所有成员属性和成员方法（包括 private 成员和静态成员）。具体是如何实现的呢？通过反编译字节码看个究竟。

先对 Outer 类进行编译 `javac Outer.java`  ，编译器在编译的时候，会将成员内部类 Inner 单独编译成一个字节码文件 `Outer$Inner.class` 。

反编译 `Outer$Inner.class`  文件得到下面的信息：

```java
E:\project\JavaExample\src\innerclassexample>javap -c Outer$Inner.class
Compiled from "Outer.java"
class innerclassexample.Outer$Inner {
  final innerclassexample.Outer this$0;

  innerclassexample.Outer$Inner(innerclassexample.Outer);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #2                  // Field this$0:Linnerclassexample/Outer;
       5: aload_0
       6: invokespecial #3                  // Method java/lang/Object."<init>":()V
       9: aload_1
      10: ldc           #4                  // String wuzy
      12: invokestatic  #5                  // Method innerclassexample/Outer.access$002:(Linnerclassexample/Outer;Ljava/lang/String;)Ljava/lang/String;
      15: pop
      16: return

  static void access$100(innerclassexample.Outer$Inner);
    Code:
       0: aload_0
       1: invokespecial #1                  // Method displayName:()V
       4: return
}
```

可以看到这两行关键信息

```java
final innerclassexample.Outer this$0;

innerclassexample.Outer$Inner(innerclassexample.Outer);
```

这就很明显了，编译器会默认为成员内部类添加了一个指向外部类对象的引用，这个引用的赋值默认是在构造函数中进行。因此可以在成员内部类中任意的访问外部类的成员。

此外也说明了成员内部类是依赖于外部类的，如果没有创建外部类，则无法对 `Outer this$0` 引用赋值，也就无法创建内部类的对象了。

## 匿名内部类

匿名内部类也就是没有名字的内部类，通常用来简化代码。

使用匿名内部类的前提条件：必须继承一个父类或者实现一个接口。形式如下：

```java
public class Outer {

    private void test(final int b) {
        final int a = 1;
        new Thread() {
            @Override
            public void run() {
                super.run();
                System.out.println(a);  
                System.out.println(b);
            }
        }.start();
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.test(2);
    }
}
```

在 jdk 1.8 之前，匿名内部类访问方法局部变量或方法形参时，局部变量和形参必须以 `final` 修饰。

为什么？

以下是分析过程。

当 外部类的 `test` 方法执行完毕，局部变量 a 和 形参 b 的都会出栈，生命周期也就结束了，但此时 Thread 对象的生命周未必就结束了，那么 `run` 方法中访问 a 或者 b 就不可能了，但是又要实现这种效果，Java 采取了 **复制** 的手段解决了这个问题。

对以上代码进行编译，编译器会将匿名内部类编译成 `Outer$1.class` 文件，再对这个字节码文件进行反编译。

```java
E:\project\JavaExample\src\innerclassexample>javap -c Outer$1.class
Compiled from "Outer.java"
class innerclassexample.Outer$1 extends java.lang.Thread {
  final int val$b;

  final innerclassexample.Outer this$0;

  innerclassexample.Outer$1(innerclassexample.Outer, int);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #1                  // Field this$0:Linnerclassexample/Outer;
       5: aload_0
       6: iload_2
       7: putfield      #2                  // Field val$b:I
      10: aload_0
      11: invokespecial #3                  // Method java/lang/Thread."<init>":()V
      14: return

  public void run();
    Code:
       0: aload_0
       1: invokespecial #4                  // Method java/lang/Thread.run:()V
       4: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
       7: iconst_1
       8: invokevirtual #6                  // Method java/io/PrintStream.println:(I)V
      11: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
      14: aload_0
      15: getfield      #2                  // Field val$b:I
      18: invokevirtual #6                  // Method java/io/PrintStream.println:(I)V
      21: return
}
```

在 `run` 方法中有一条指令：

```java
iconst_1
```

这条指令表示将操作数 1 压入栈中（当 int 取值-1~5 时，JVM 采用 `iconst` 指令将常量压入栈中），表示使用的是一个本地局部变量。

还有这三行信息：

```java
  final int val$b;

  final innerclassexample.Outer this$0;

  innerclassexample.Outer$1(innerclassexample.Outer, int);
```

`this$0` 是指向外部类的引用，`val$b` 是形参 b 的拷贝，都是由编译器在构造函数中赋值初始化的。

从上面可以看出，如果局部变量的值在编译期间就可以确定，则直接在匿名类内部里面创建一个拷贝，如果局部变量的值无法在编译期间确定，则通过构造器传参的方式来对拷贝进行初始化赋值。

**这就导致了一个新的问题，数据不一致**。`run` 方法访问的 a 压根不是 `test` 方法的局部变量 a，当在 `run` 方法改变变量 a 时候，`test` 方法的局部变量 a 并没有改变。

为了解决这个问题，Java 采取了粗暴的方式，限定必须将变量 a 限制为 final 变量，不允许对变量 a 进行更改（对于引用类型的变量，是不允许指向新的对象），这样数据不一致性的问题就得以解决了。

这也就解释了为什么匿名内部类只能访问局部 final 变量了。

在 JDK 1.8 以后，匿名内部类可以访问到非 final 变量了。以下这种写法完全没问题。

```java
public class Outer {

    private void test(int b) {
        int a = 1;
        new Thread() {
            @Override
            public void run() {
                super.run();
                System.out.println(a);
                System.out.println(b);
            }
        }.start();
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.test(2);
    }
}

```

对其反编译下：

```java

E:\project\JavaExample\src\innerclassexample>javap -c Outer$1.class
Compiled from "Outer.java"
class innerclassexample.Outer$1 extends java.lang.Thread {
  final int val$a;

  final int val$b;

  final innerclassexample.Outer this$0;

  innerclassexample.Outer$1(innerclassexample.Outer, int, int);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #1                  // Field this$0:Linnerclassexample/Outer;
       5: aload_0
       6: iload_2
       7: putfield      #2                  // Field val$a:I
      10: aload_0
      11: iload_3
      12: putfield      #3                  // Field val$b:I
      15: aload_0
      16: invokespecial #4                  // Method java/lang/Thread."<init>":()V
      19: return

  public void run();
    Code:
       0: aload_0
       1: invokespecial #5                  // Method java/lang/Thread.run:()V
       4: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
       7: aload_0
       8: getfield      #2                  // Field val$a:I
      11: invokevirtual #7                  // Method java/io/PrintStream.println:(I)V
      14: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      17: aload_0
      18: getfield      #3                  // Field val$b:I
      21: invokevirtual #7                  // Method java/io/PrintStream.println:(I)V
      24: return
}
```

可以看到这四行

```java
final int val$a;

final int val$b;

final innerclassexample.Outer this$0;

innerclassexample.Outer$1(innerclassexample.Outer, int, int);
```

JVM 编译器会在匿名内部类的构造函数中对局部变量 a 和 形参 b 进行拷贝赋值。而且， `run` 方法是无法修改变量 a 和 形参 b 的值的。

## 局部内部类

定义在方法体或者代码块里的类称为局部内部类。形式如下：

```java
public class Outer {

    private int num1 = 1;

    private void test() {
        class Inner {
            private int num2 = 2;
            private void display() {
                System.out.println(num1);
                System.out.println(num2);
            }
        }
        Inner inner = new Inner();
        inner.display();
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.test();
    }
}
```

注意，局部内部类就像是方法里面的一个局部变量一样，是不能有public、protected、private以及static修饰符的。

## 静态内部类

静态内部类与成员内部类的定义方式类似，也是定义在另一个类的内部，只不过在类的前面多了一个 `static` 关键字。形式如下：

```java
public class Outer {

    static class Inner {
    }
}
```

静态内部类与类的静态属性类似，不依赖于对象，无法访问外部类的非静态成员，因为外部类的非静态成员依附于具体的对象。从下面的反编译结果也能看出，静态内部类是不持有外部类对象的引用的。

```jav
E:\project\JavaExample\src\innerclassexample>javap -c Outer$Inner.class
Compiled from "Outer.java"
class innerclassexample.Outer$Inner {
  innerclassexample.Outer$Inner();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return
}
```

## 内部类的使用场景和好处

1、内部类使得多继承的解决方案变得完整。内部类(除去用 static 修饰的 )可以直接使用其外部类的成员变量以及成员函数，达到一个继承的效果，再加上自身继承基类来达到一个多重继承的效果。

2、方便将存在一定逻辑关系的类组织在一起，又可以对外界隐藏。

3、方便编写事件驱动程序。比如 Android 里面的事件监听。

## 参考

https://www.cnblogs.com/dolphin0520/p/3811445.html
