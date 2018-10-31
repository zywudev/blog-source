---
title: Android Clean 架构
date: 2018-10-16 14:27:00
tags:
---

 Clean 一般是指，代码以洋葱的形状依据一定的依赖规则被划分为多层：内层对于外层一无所知。这就意味着**依赖只能由外向内**。

![](https://github.com/zywudev/blog-source/raw/master/image/FneK1RrTl0mCEVOQydNJCXJiv00v.png)

**Clean 架构的准则：**

- 架构独立。架构不依赖于一些满载功能的软件库。
- 可测试性。业务规则可以在没有 UI、数据库等外部元素的情况下完成测试。
- UI的独立性。在不改变系统其余部分的情况下完成UI的简易修改。
- 数据库的独立性。业务规则不绑定数据库，可以随意更换数据库的实现。
- 外部机制独立。业务规则不知道外层的事情。

### Framework and Drivers

框架或者驱动，包括 UI、框架、数据库实现、网络实现细节等。

### Interface Adapter

接口适配层，负责将实现细节和业务逻辑连接起来的粘合层。

### Business rules(Interactors)

业务规则，整合了实现系统需要的所有实例。

### Domain logic

封装了业务实体。实体可以是包含有方法的对象，或者一系列的数据结构、函数。

依据这些规则将工程分为三层：

![](https://github.com/zywudev/blog-source/blob/master/image/FiHNW7VqAGM0mduIOUbZJT3waAu6.png)

### Presentation Layer

MVC 或者 MVP 对应的地方，不处理 UI 以外的任何逻辑。 

### Domain Layer

业务逻辑 Use Case 实现的地方。属于系统最内层。

这一层为纯 Java 代码，不牵扯任何 Android 相关依赖，规定了要做什么，具体实现细节交给外层。

### Data Layer

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/FhHTuUPyCcYBna7CGmYtsMG5OpLO.png)

所有系统需要的数据通过这一层的 Repository 获取， 这是一种 Repository 模式，具体看[这里](https://docs.microsoft.com/en-us/previous-versions/msp-n-p/ff649690(v=pandp.10))。Repository 接口定义是在 Domain 层，接口表示怎么去存储或者访问数据，这些是业务逻辑，但是具体的实现与业务逻辑无关，应该交给 Data 层。

### 总结

1、Clean 架构中内层意味着抽象，外层意味着细节，同样一个抽象可能有多个子类，这种一对多的方式更具灵活性。

2、细节依赖抽象，业务逻辑制定规则，外层实现接口，这样能保证在内层能够调用外层组件去实现需要的逻辑，这里依据的是 DIP。

3、Clean 架构较为繁琐，如果是简单项目，完全没必要使用。

### 测试方法

- Presentation Layer： 使用 AndroidInstruction 和 espresso 做集成测试和功能测试
- Domain Layer：使用 JUnit 和 mockito 做单元测试
- Data Layer：使用 Robolectric（这层有Android依赖）和 junit、mockito 做集成和单元测试。

### 学习项目

- [https://github.com/android10/Android-CleanArchitecture](https://github.com/android10/Android-CleanArchitecture)
- [https://github.com/crazysunj/CrazyDaily](https://github.com/crazysunj/CrazyDaily)

