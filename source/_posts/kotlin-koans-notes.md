---
title: Kotlin Koans 学习笔记
date: 2019-06-28 17:07:12
tags:
- Android
- Kotlin
categories: Kotlin
---

> [Kotlin Koans](https://github.com/Kotlin/kotlin-koans) 是 Kotlin 官方推出的一系列 Kotlin 语法练习。
>
> 一共分为 6 个模块，每个模块若干任务，每个任务都有一系列单元测试，目标就是编码通过单元测试。
>
> 本文是在学习 Kotlin Koans 过程中将相关语法点做一个简单的记录。

# Introduction

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

# Collections

这一模块主要是集合的一些任务，所有任务都是围绕一个商店（Shop）展开，商店有一个客户（Customer）列表。

客户具有姓名、城市和订单（Order）列表三个属性。

订单具有商品（Product）列表和是否已经发货两个属性。

商品具有名称和价格两个属性。

```kotlin
data class Shop(val name: String, val customers: List<Customer>)

data class Customer(val name: String, val city: City, val orders: List<Order>) {
    override fun toString() = "$name from ${city.name}"
}

data class Order(val products: List<Product>, val isDelivered: Boolean)

data class Product(val name: String, val price: Double) {
    override fun toString() = "'$name' for $price"
}

data class City(val name: String) {
    override fun toString() = name
}
```

## Introduction

要求返回一个 `Set`，包含商店中所有的客户：

```kotlin
fun Shop.getSetOfCustomers(): Set<Customer> {
    // Return a set containing all the customers of this shop
    return customers.toSet()
//    return this.customers
}
```

## Filter Map

`filter` 方法返回一个包含所有满足指定条件元素的列表。

任务第一个要求返回指定城市所有客户的列表。

```kotlin
fun Shop.getCustomersFrom(city: City): List<Customer> {
    // Return a list of the customers who live in the given city
    return customers.filter { it.city == city }
}
```

`map` 方法将指定的转换函数运用到原始集合的每一个元素，并返回一个转换后的集合。

任务第二个要求返回所有客户所在城市的 Set。

```kotlin
fun Shop.getCitiesCustomersAreFrom(): Set<City> {
    // Return the set of cities the customers are from
    return customers.map { it.city }.toSet()
}
```

## All Any and others Predicates

`all`：如果所有的元素都满足指定的条件返回 true。

`any`：如果至少有一个元素满足指定的条件返回 ture。

`count`：返回满足指定条件的元素数量。

`firstOrAll`：返回第一个满足指定条件的元素，如果没有就返回 null。

```kotlin
fun Customer.isFrom(city: City): Boolean {
    // Return true if the customer is from the given city
    return this.city == city
}

fun Shop.checkAllCustomersAreFrom(city: City): Boolean {
    // Return true if all customers are from the given city
    return customers.all { it.isFrom(city) }
}

fun Shop.hasCustomerFrom(city: City): Boolean {
    // Return true if there is at least one customer from the given city
    return customers.any { it.isFrom(city) }
}

fun Shop.countCustomersFrom(city: City): Int {
    // Return the number of customers from the given city
    return customers.count { it.isFrom(city) }
}

fun Shop.findFirstCustomerFrom(city: City): Customer? {
    // Return the first customer who lives in the given city, or null if there is none
    return customers.firstOrNull { it.isFrom(city) }
}
```

## FlatMap

`flatmap`：针对列表中的每一项根据指定的方法生成一个列表，最后将所有的列表拼接成一个列表返回。

```kotlin
// 返回一个客户所有已订购的产品
val Customer.orderedProducts: Set<Product>
    get() {
        // Return all products this customer has ordered
        return orders.flatMap { it.products }.toSet()
    }
// 返回所有至少被一个客户订购过的商品集合
val Shop.allOrderedProducts: Set<Product>
    get() {
        // Return all products that were ordered by at least one customer
        return customers.flatMap { it.orderedProducts }.toSet()
    }
```

## Max Min

`max`：返回集合中最大的元素。如果没有元素则返回 null。

`maxby`：使用函数参数计算的值作为比较对象。返回最大的元素中的值。

```kotlin
// 返回商店中订单数目最多的一个客户。
fun Shop.getCustomerWithMaximumNumberOfOrders(): Customer? {
    // Return a customer whose order count is the highest among all customers
    return customers.maxBy { it.orders.size }
}

// 返回一个客户所订购商品中价格最高的一个商品
fun Customer.getMostExpensiveOrderedProduct(): Product? {
    // Return the most expensive product which has been ordered
    return orders.flatMap { it.products }.maxBy { it.price }
}
```

## Sort

kotlin 中可以通过 `sortby`指定排序的标准。

任务要求返回一个客户列表，客户的顺序是根据订单的数量由低到高。

```kotlin
fun Shop.getCustomersSortedByNumberOfOrders(): List<Customer> {
    // Return a list of customers, sorted by the ascending number of orders they made
    return customers.sortedBy { it.orders.size }
}
```

## Sum

`sumby`：将集合中所有元素按照指定的函数变换以后的结果累加。

任务要求计算一个客户所有已订购商品的价格总和。

```kotlin
fun Customer.getTotalOrderPrice(): Double {
    // Return the sum of prices of all products that a customer has ordered.
    // Note: a customer may order the same product several times.
    return orders.flatMap { it.products }.sumByDouble { it.price }
}
```

## GroupBy

`groupBy`方法返回一个根据指定条件分组好的 map。

任务要求返回来自每一个城市的客户的 map：

```kotlin
fun Shop.groupCustomersByCity(): Map<City, List<Customer>> {
    // Return a map of the customers living in each city
    return customers.groupBy { it.city }
}
```

## Partition

`partition`：将原始的集合分为一对集合，这一对集合中第一个是满足指定条件的元素集合，第二个是不满足指定条件的集合。

任务要求返回所有未发货订单数目多于已发货订单的用户。

```kotlin
fun Shop.getCustomersWithMoreUndeliveredOrdersThanDelivered(): Set<Customer> {
    // Return customers who have more undelivered orders than delivered
    return customers.filter {
        val (delivered, undelivered) = it.orders.partition { it.isDelivered }
        delivered.size < undelivered.size
    }.toSet()
}
```

## Fold

`fold`：给定一个初始值，然后通过迭代对集合中的每一个元素执行指定的操作并将操作的结果累加。

任务要求返回所有客户都订购过的商品。

```kotlin
fun Shop.getSetOfProductsOrderedByEachCustomer(): Set<Product> {
    // Return the set of products that were ordered by each of the customers
    return customers.fold(allOrderedProducts, { orderedByAll, customer ->
        orderedByAll.intersect(customer.orderedProducts)
    })
```

## CompoundTasks

上述方法的混合使用。

```kotlin

fun Customer.hasOrderedProduct(product: Product) = orders.any { it.products.contains(product) }

// 所有购买了指定商品的客户列表
fun Shop.getCustomersWhoOrderedProduct(product: Product): Set<Customer> {
    // Return the set of customers who ordered the specified product
    return customers.filter { it.hasOrderedProduct(product) }.toSet()
}

// 查找某个用户所有已发货的商品中最昂贵的商品
fun Customer.getMostExpensiveDeliveredProduct(): Product? {
    // Return the most expensive product among all delivered products
    // (use the Order.isDelivered flag)
    return orders.filter { it.isDelivered }.flatMap { it.products }.maxBy { it.price }
}

// 查找指定商品被购买的次数
fun Shop.getNumberOfTimesProductWasOrdered(product: Product): Int {
    // Return the number of times the given product was ordered.
    // Note: a customer may order the same product for several times.
    return customers.flatMap { it.orders.flatMap { it.products } }.count { it == product }
}
```

## Extensions On Collections

重写 Java 代码

```java
public Collection<String> doSomethingStrangeWithCollection(Collection<String> collection) {
    Map<Integer, List<String>> groupsByLength = Maps.newHashMap();
    for (String s : collection) {
        List<String> strings = groupsByLength.get(s.length());
        if (strings == null) {
            strings = Lists.newArrayList();
            groupsByLength.put(s.length(), strings);
        }
        strings.add(s);
    }

    int maximumSizeOfGroup = 0;
    for (List<String> group : groupsByLength.values()) {
        if (group.size() > maximumSizeOfGroup) {
            maximumSizeOfGroup = group.size();
        }
    }

    for (List<String> group : groupsByLength.values()) {
        if (group.size() == maximumSizeOfGroup) {
            return group;
        }
    }
    return null;
}
```

方法的目的是：

- 将一个字符串集合按照长度分组，放入一个 map 中
- 求出 map 中所有元素 (String List) 的最大长度
- 根据步骤2的结果，返回 map 中字符串数目最多的那一组

```kotlin
fun doSomethingStrangeWithCollection(collection: Collection<String>): Collection<String>? {
    val groupsByLength = collection.groupBy { s -> s.length }
    return groupsByLength.values.maxBy { group -> group.size }
}
```

# Conventions

## Comparsion

实现日期对象大小的比较

```kotlin
fun task25(date1: MyDate, date2: MyDate): Boolean {
    return date1 < date2
}
```

其中类 `MyDate`需要实现 `Comparable`接口。

```kotlin
data class MyDate(val year: Int, val month: Int, val dayOfMonth: Int) : Comparable<MyDate> {
    override fun compareTo(other: MyDate) =
            when {
                other.year != year -> year - other.year
                other.month != month -> month - other.month
                else -> dayOfMonth - other.dayOfMonth
            }

}
```

## InRange

实现检查指定的日期是不是在某一个日期范围内。

```kotlin
fun checkInRange(date: MyDate, first: MyDate, last: MyDate): Boolean {
    return date in DateRange(first, last)
}
```

DateRange 类已经定义好，需要添加 `contains`函数。

```kotlin
class DateRange(val start: MyDate, val endInclusive: MyDate) {
    operator fun contains(value: MyDate): Boolean = start <= value && value <= endInclusive
}
```

## RangeTo

这一个任务是要求实现`MyDate`类的`..`运算符。`..`运算符最终会翻译成`rangeTo()`函数，所以本任务就是实现`MyDate.rangeTo()`。

```kotlin
operator fun MyDate.rangeTo(other: MyDate): DateRange = DateRange(this, other)

fun checkInRange2(date: MyDate, first: MyDate, last: MyDate): Boolean {
    //todoTask27()
    return date in first..last
}
```

## ForLoop

任务要求对 DateRange 内的 MyDate 执行 for 循环，因此 DateRange 需要实现 Iterable 接口。

```kotlin
class DateRange(val start: MyDate, val endInclusive: MyDate) : Iterable<MyDate> {
    override fun iterator(): Iterator<MyDate> = DateIterator(this)

    operator fun contains(value: MyDate): Boolean = start <= value && value <= endInclusive
}

class DateIterator(val dateRange: DateRange) : Iterator<MyDate> {
    var current: MyDate = dateRange.start
    override fun hasNext(): Boolean = current <= dateRange.endInclusive

    override fun next(): MyDate {
        val result = current
        current = current.nextDay()
        return result
    }
}
```

## OperatorOverloading

重载 `+`运算符：

```kotlin
operator fun MyDate.plus(timeInterval: TimeInterval) = addTimeIntervals(timeInterval, 1)
```

重载`*`运算符：

```kotlin
// 将TimeInterval的乘法结果定义成RepeatedTimeInterval
class RepeatedTimeInterval(val timeInterval: TimeInterval, val number: Int)
operator fun TimeInterval.times(number: Int) = RepeatedTimeInterval(this, number)

operator fun MyDate.plus(timeIntervals: RepeatedTimeInterval) = addTimeIntervals(timeIntervals.timeInterval, timeIntervals.number)

```

## Destructuring Declaration

kotlin 可以将一个对象的所有属性一次赋值给一堆变量

任务要求判断一个日期是否是闰年。

```kotlin
data class MyDate(val year: Int, val month: Int, val dayOfMonth: Int)

fun isLeapDay(date: MyDate): Boolean {
    val (year, month, dayOfMonth) = date

    // 29 February of a leap year
    return isLeapYear(year) && month == 1 && dayOfMonth == 29
}

// Years which are multiples of four (with the exception of years divisible by 100 but not by 400)
fun isLeapYear(year: Int): Boolean = year % 4 == 0 && (year % 100 != 0 || year % 400 == 0)
```

## Invoke

如果一个类实现了`invoke`函数，那么该类的实体对象在调用这个函数时可以省略函数名，`invoke`函数需要有`operator`修饰符。

```kotlin
fun task31(invokable: Invokable): Int {
    return invokable()()()().numberOfInvocations
}

class Invokable {
    var numberOfInvocations: Int = 0
    operator fun invoke(): Invokable {
        numberOfInvocations++
        return this
    }
}
```

# Properties

## Properties

kotlin 中可以为属性自定义 setter，格式如下：

```kotlin
class PropertyExample() {
    var counter = 0
    var propertyWithCounter: Int? = null
        set(value) {
            field = value
            counter++
        }
}
```

每次为 `propertyWithCounter` 赋值时，都会调用 `set`，kotlin 中不能直接声明字段，然而，当一个属性需要一个幕后字段时，Kotlin 会自动提供。这个幕后字段可以使用`field`标识符在访问器中引用。

## LazyProperty

`initializer`是一个`lambda `表达式，这个表达式会在`lazy`属性第一次被访问的时候执行，且仅执行一次

```kotlin
class LazyProperty(val initializer: () -> Int) {
    var value: Int? = null
    val lazy: Int
    get() {
        if (value == null) {
            value = initializer()
        }
        return value!!
    }
}
```

## DelegatesExamples

用委托实现懒加载

```kotlin
class LazyPropertyUsingDelegates(val initializer: () -> Int) {
    val lazyValue: Int by lazy(initializer)
}
```

## HowDelegatesWork

```kotlin
class EffectiveDate<R> : ReadWriteProperty<R, MyDate> {
    var timeInMillis: Long? = null

    operator override fun getValue(thisRef: R, property: KProperty<*>): MyDate = timeInMillis!!.toDate()
    operator override fun setValue(thisRef: R, property: KProperty<*>, value: MyDate) { timeInMillis = value.toMillis() }
}
```

# Builders

## ExtensionFunctionLiterals

```kotlin
fun task36(): List<Boolean> {
    val isEven: Int.() -> Boolean = { this % 2 == 0 }
    val isOdd: Int.() -> Boolean = { this % 2 != 0 }

    return listOf(42.isOdd(), 239.isOdd(), 294823098.isEven())
}
```

## StringAndMapBuilders

类型扩展函数，仿照`buildString`实现`buildMap`

```kotlin
fun <K, V> buildMap(build: MutableMap<K, V>.() -> Unit): Map<K, V> {
    val map = HashMap<K, V>()
    map.build()
    return map
}

fun task37(): Map<Int, String> {
    return buildMap {
        put(0, "0")
        for (i in 1..10) {
            put(i, "$i")
        }
    }
}
```

## TheFunctionApply

使用`apply`重写上一练习中的功能

```kotlin
fun <T> T.myApply(f: T.() -> Unit): T {
    f()
    return this
}
```

## HtmlBuilders

把`products`填充进表格，并设置好背景色，运行`htmlDemo.kt`可以预览内容

```kotlin
fun renderProductTable(): String {
    return html {
        table {
            tr {
                td {
                    text("Product")
                }
                td {
                    text("Price")
                }
                td {
                    text("Popularity")
                }
            }
            val products = getProducts()
            for ((index, product) in products.withIndex()) {
                tr {
                    td (color = getCellColor(index, 0)) {
                        text(product.description)
                    }
                    td (color = getCellColor(index, 1)) {
                        text(product.price)
                    }
                    td (color = getCellColor(index, 2)) {
                        text(product.popularity)
                    }
                }
            }
        }
    }.toString()
}

```

## BuildersHowItWorks

1：c，`td`是一个方法，这里的`td`显然是在调用
2：b，`color`是参数名，这里使用了命名参数
3：b，这里是个`lambda`表达式
4：c，`this`指向调用者

# Generics

## GenericsFunctions

根据条件将一个集合分为两个集合。

```kotlin
fun <T, C: MutableCollection<T>> Collection<T>.partitionTo(first: C, second: C, predicate: (T) -> Boolean): Pair<C, C> {
    for (element in this) {
        if (predicate(element)) {
            first.add(element)
        } else {
            second.add(element)
        }
    }
    return Pair(first, second)
}
```

