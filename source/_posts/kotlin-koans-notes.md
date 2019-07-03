---
title: Kotlin Koans 学习笔记
date: 2019-06-28 17:07:12
tags:
- Android
- Kotlin
---

> [Kotlin Koans](https://github.com/Kotlin/kotlin-koans) 是 Kotlin 官方推出的一系列 Kotlin 语法练习。
>
> 一共分为 6 个模块，每个模块若干任务，每个任务都有一系列单元测试，目标就是编码通过单元测试。
>
> 本文是在学习 Kotlin Koans 过程中将相关语法点做一个简单的记录。

## Hello_World

和大多数语言一样，第一个任务的名称就是 Hello World。这个任务很简单，就是要求 task0 函数返回一个字符串`ok`。

```kotlin
fun task0(): String {
    return "OK"
}
```

Kotlin 中函数使用关键字`fun`声明，返回类型在函数名称的后面，中间以`:`分开。

## Java to Kotlin Convert

将 Java 代码转化为 Kotlin 代码。

```kotlin
// java
public String task1(Collection<Integer> collection) {
    StringBuilder sb = new StringBuilder();
    sb.append("{");
    Iterator<Integer> iterator = collection.iterator();
    while (iterator.hasNext()) {
        Integer element = iterator.next();
        sb.append(element);
        if (iterator.hasNext()) {
            sb.append(", ");
        }
    }
    sb.append("}");
    return sb.toString();
}

//kotlin
fun task1(collection: Collection<Int>): String {
    val sb = StringBuilder()
    sb.append("{")
    val iterator = collection.iterator()
    while (iterator.hasNext()) {
        val element = iterator.next()
        sb.append(element)
        if (iterator.hasNext()) {
            sb.append(", ")
        }
    }
    sb.append("}")
    return sb.toString()
}
```

## Named Arguments

使用 kotlin 提供的 `joinToString()` 完成任务1，只指定 `joinToString()` 的参数。

kotlin 中函数参数可以有默认值，当省略相应的参数时使用默认值。与其他语言相比，这可以减少重载数量：

默认值通过类型后面的`=`及给出的值定义。

```kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) { …… }
```

重写一个有默认参数的函数时，我们不允许重新指定默认参数的值。

```kotlin
open class A {
    open fun foo(i: Int = 10) { …… }
}

class B : A() {
    override fun foo(i: Int) { …… }  // 不能有默认值
}
```

可以在调用函数时使用命名的函数参数。当一个函数有大量的参数或默认参数时这会非常方便。

回到任务本身，先看一下函数 `joinToString`：

```kotlin
public fun <T> Iterable<T>.joinToString(separator: CharSequence = ", ", prefix: CharSequence = "", postfix: CharSequence = "", limit: Int = -1, truncated: CharSequence = "...", transform: ((T) -> CharSequence)? = null): String {
    return joinTo(StringBuilder(), separator, prefix, postfix, limit, truncated, transform).toString()
}
```

该函数对分隔符，前缀，后缀等其他参数都指定了默认值，我们只需要重新指定前缀、后缀两个参数。

```kotlin
fun task2(collection: Collection<Int>): String {
    return collection.joinToString(postfix = "}",prefix = "{")
}
```

## Default Arguments

使用参数默认值，修改`foo`函数，实现 Java 中需要重载才能实现的功能。

```kotlin
fun foo(name: String, number: Number = 42, toUpperCase: Boolean = false): String {
    return (if (toUpperCase) name.toUpperCase() else name) + number
}

fun task3(): String {

    return (foo("a") +
            foo("b", number = 1) +
            foo("c", toUpperCase = true) +
            foo(name = "d", number = 2, toUpperCase = true))
}
```

非常简洁。这里还可以看出 `if` 是一个表达式，即它会返回一个值。 因此就不需要三元运算符（条件 ? 然后 : 否则）

## Lambdas

使用 Lambda 重写 Java 代码：

```kotlin
// java
public boolean task4(Collection<Integer> collection) {
     return Iterables.any(collection, new Predicate<Integer>() {
         @Override
         public boolean apply(Integer element) {
             return element % 2 == 0;
         }
     });
 }

// kotlin
fun task4(collection: Collection<Int>): Boolean = collection.any { x -> x % 2 == 0 }
```

Lambda 表达式的特征：

- Lambdas 表达式是花括号括起来的代码块。
- 如果一个 lambda 表达式有参数，前面是参数，后跟`->` 。
- 函数体写在`->`符号后面。

## String Templates

生成一个正则表达式，可以匹配`13 JUN 1992` 这样格式的字符串。

kotlin 中支持两种字符串格式：

一种是一对`"`包起来的字符串，支持转义字符。

```kotlin
val s = "Hello, world!\n"
```

一种是一对` """`包起来的字符串，不需要用`\`来转义，可以直接使用各种符号，包括换行符。

```
val text = """
    for (c in "foo")
        print(c)
"""
```

字符串字面值可以包含*模板表达式* ，即一些小段代码，会求值并把结果合并到字符串中。 模板表达式以美元符（`$`）开头，由一个简单的名字构成:

```kotlin
val i = 10
println("i = $i") // 输出“i = 10”
```

回到任务，修改模板字符串，使其可以匹配测试中的日期格式。

```kotlin
val month = "(JAN|FEB|MAR|APR|MAY|JUN|JUL|AUG|SEP|OCT|NOV|DEC)"
fun task5(): String = """\d{2}\ $month\ \d{4}"""
```

## Data Classes

将 Java 中的数据实体类转化成 Kotlin。

```kotlin
// java
public static class Person {
     private final String name;
     private final int age;

     public Person(String name, int age) {
         this.name = name;
         this.age = age;
     }

     public String getName() {
         return name;
     }

     public int getAge() {
         return age;
     }
 }

// kotlin
data class Person(val name: String, val age: Int)
```

kotlin 中使用`data`标记的类，编译器会自动根据主构造函数中定义的属性生成下面这些成员函数：

- `equals()`/`hashCode()` 对；
- `toString()` 格式是 `"Person(name=John, age=42)"`；
- `componentN()`按声明顺序对应于所有属性；
- `copy()` 函数。

## Nullable Type

将 Java 版本的 `sendMessageToClient` 方法使用 kotlin 改写，只能使用 `if` 表达式。

```java
public void sendMessageToClient(@Nullable Client client, @Nullable String message, @NotNull Mailer mailer) {
    if (client == null || message == null) return;

    PersonalInfo personalInfo = client.getPersonalInfo();
    if (personalInfo == null) return;

    String email = personalInfo.getEmail();
    if (email == null) return;

    mailer.sendMessage(email, message);
}
```

kotlin 中，如果一个变量可能为空，在定义的时候需要在类型后面加上 `?` 

```kotlin
val a: Int? = null
```

对于可能为空的变量，如果必须在其值不为 null 时才进行后续操作，可以使用 `?.`操作符进行保护。

```kotlin
a?.toLong()
```

如果变量为空，想要提供一个替代值，可以使用 `?:`：

```kotlin
val b = a?.toLong() ?: 0L
```

回到任务本身。

```kotlin
fun sendMessageToClient(
        client: Client?, message: String?, mailer: Mailer
) {
    val email = client?.personalInfo?.email
    if (email != null && message != null) {
        mailer.sendMessage(email,message)
    }
}
```

## Smart Casts

使用 kotlin 中 的Smart Cast 和 when表达式重新实现 Java 代码中的 `eval()` 函数。

```java
 public int eval(Expr expr) {
     if (expr instanceof Num) {
         return ((Num) expr).getValue();
     }
     if (expr instanceof Sum) {
         Sum sum = (Sum) expr;
         return eval(sum.getLeft()) + eval(sum.getRight());
     }
     throw new IllegalArgumentException("Unknown expression");
 }
```

kotlin 中使用`when`代替了`switch`，功能更强大。

在 `when` 中，我们使用 `->` 表示执行某一分支后的操作，`->` 之前是条件，可以是任何表达式。

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}

when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

回到任务本身。

```kotlin
fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.left) + eval(e.right)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

## Extension Functions

kotlin 支持为已经存在的类编写扩展方法，而且不需要对原有类进行任何改动，只需要使用`className.extensionFun`的形式就可以了

```kotlin
fun String.lastChar() = this.get(this.length - 1)
```

任务要求为`Int`和`Pair<Int, Int>`分别实现一个扩展函数`r()`。`r()`函数的功能就是创建一个有理数。

```kotlin
data class RationalNumber(val numerator: Int, val denominator: Int)

fun Int.r(): RationalNumber = RationalNumber(this, 1)
fun Pair<Int, Int>.r(): RationalNumber = RationalNumber(this.first, this.second)
```

## Object Expression

任务的要求是创建一个比较器（comparator），提供给 Collection 类对 list 按照降序排序。

```kotlin
fun task10(): List<Int> {
    val arrayList = arrayListOf(1, 5, 2)
    Collections.sort(arrayList, object : Comparator<Int> {
        override fun compare(o1: Int, o2: Int): Int {
            return o2 - o1
        }
    })
    return arrayList
}
```

## SAM Conversions

SAM conversions 就是如果一个 object 实现了一个 SAM（Single Abstract Method）接口时，可以直接传递一个 lambda 表达式。

```kotlin
fun task11(): List<Int> {
    val arrayList = arrayListOf(1, 5, 2)
    arrayList.sortWith(Comparator { x, y -> y - x })
    return arrayList
}
```

## Extensions On Collections

使用扩展函数`sortedDescending`重写上一个任务中的代码：

```kotlin
fun task12(): List<Int> {
    return arrayListOf(1, 5, 2).sortedDescending()
}
```

