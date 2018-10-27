---
title: Android 单元测试 Junit 
date: 2018-10-24 15:15:38
tags:
---

> 示例代码：https://github.com/zywudev/AndroidUnitTest

Anroid Studio 在新建项目中自动将 Junit 框架集成，无需额外导入依赖。

## Junit Assert

Assert 就是断言，判断假设与实际是否一致，一致则测试通过。

常用断言：

- assertTrue  假设为真
- assertFalse 假设为假
- assertEquals 假设相同（基本数据类型或者对象）
- assertNotEquals 假设不相同（基本数据类型或者对象）
- assertNull 假设为空
- assertNotNull 假设不为空
- assertSame 假设相同（只能是对象）
- assertNotSame 假设不相同（只能是对象）
- assertArrayEquals 假设数组相同
- assertThat 断言实际值是否满足指定的条件

期望值是前一个参数，实际值是后一个参数。

**assertThat**

```java
assertThat(T actual, Matcher<? super T> matcher);

assertThat(String reason, T actual, Matcher<? super T> matcher); 
```

其中，reason 为断言失败的输出信息，actual 为实际值，matcher 为匹配器。

常用的匹配器整理：

- is	断言参数等于后面给出的匹配表达式	
- not	断言参数不等于后面给出的匹配表达式	
- equalTo	断言参数相等	
- equalToIgnoringCase	断言字符串相等忽略大小写
- containsString	断言字符串包含某字符串	
- startsWith	断言字符串以某字符串开始
- endsWith	断言字符串以某字符串结束
- nullValue	断言参数的值为null
- notNullValue	断言参数的值不为null	
- greaterThan	断言参数大于	
- lessThan	断言参数小于
- greaterThanOrEqualTo	断言参数大于等于
- lessThanOrEqualTo	断言参数小于等于
- closeTo	断言浮点型数在某一范围内
- allOf	断言符合所有条件，相当于&&	
- anyOf	断言符合某一条件，相当于或
- hasKey	断言Map集合含有此键	
- hasValue	断言Map集合含有此值	
- hasItem	断言迭代对象含有此元素

自定义匹配器：

```java
/**
 * Created by wuzy on 2018/10/26.
 * 自定义匹配器，判断数字是否在范围内
 */

public class IsNumberRangeMatcher extends BaseMatcher<Integer> {

    private int start;

    private int end;

    public IsNumberRangeMatcher(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    public boolean matches(Object item) {
        if (item == null) {
            return false;
        }
        Integer i = (Integer) item;
        return start <= i && i <= end;
    }

    @Override
    public void describeTo(Description description) {

    }
}

```

http://www.vogella.com/tutorials/Hamcrest/article.html

## Junit Annotation

常用注解：

- @Test	表示此方法为测试方法
- @Before	在每个测试方法前执行，可做初始化操作
- @After	在每个测试方法后执行，可做释放资源操作
- @Ignore	忽略的测试方法
- @BeforeClass	在类中所有方法前运行。此注解修饰的方法必须是static void
- @AfterClass	在类中最后运行。此注解修饰的方法必须是static void
- @RunWith	指定该测试类使用某个运行器
- @Parameters	指定测试类的测试数据集合
- @Rule	重新制定测试类中方法的行为
- @FixMethodOrder	指定测试类中方法的执行顺序

执行顺序：

@BeforeClass –> @Before –> @Test –> @After –> @AfterClass

**@Test**

它可以接受两个参数，一个是预期异常，一个是超时时间。

即不出现预期异常则测试不通过；超过超时时间则测试不通过。

```java
@Test(expected = ParseException.class)   // 日期解析错误，预期异常
public void dateToStamp1() throws Exception {
    DateUtil.dateToStamp(time);
}

@Test(timeout = 100)
public void dateToStamp2() throws Exception {
    DateUtil.dateToStamp(time);
}
```

**@Parameters**

参数化测试，用于测试数据集合。

```java
@RunWith(Parameterized.class)
public class DateUtilTest {
   
    @Parameterized.Parameters
    public static Collection primeNumber() {
       return Arrays.asList("2018-10-24", "2018-10-24 16:18:28","2018年10月24日 16点18分28秒");
    }
   ...
}
```

**@Rule**

Junit 提供自定义规则。

- 实现 TestRule 接口，实现 apply 方法。

```java
/**
 * Created by wuzy on 2018/10/26.
 * 自定义 Rule，单元测试方法执行前后打印
 */
public class MyRule implements TestRule {

    @Override
    public Statement apply(final Statement base, final Description description) {
        return new Statement() {
            @Override
            public void evaluate() throws Throwable {
                // evaluate前执行方法相当于@Before
                String methodName = description.getMethodName(); // 获取测试方法的名字
                System.out.println(methodName + "测试开始！");

                base.evaluate();  // 运行的测试方法

                // evaluate后执行方法相当于@After
                System.out.println(methodName + "测试结束！");
            }
        };
    }
}
```

- 在测试类中添加自定义 Rule 。

```java
 // 添加自定义 Rule
@Rule
public MyRule myRule = new MyRule();
```

