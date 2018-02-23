---
title: Android 编码规范
date: 2017-08-28 15:34:14
tags:
---

简单总结了 Android 开发中的一些代码规范，供开发者参考。

## 0 Index

- [1 命名规范](#1-命名规范)
- [2 注释规范](#2-注释规范)
- [3 格式规范](#3-格式规范)
- [4 参考资料](#4-参考资料)

## 1 命名规范

**大驼峰命名（UpperCamelCase）**：每个单词的第一个字母都大写。

**小驼峰命名（lowerCamelCase）**：除第一个单词以外，每一个单词的第一个字母大写。

**命名的基本原则**： 

- 不能以下划线或美元符号开始，也不能以下划线或美元符号结束。
- 严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式。但比如 `shanghai` 等通用的名称，可视同英文。
- 除了常见的英文缩写，尽量避免缩写。

### 1.1 类 / 接口命名

- 使用大驼峰命名法，用名词或者名词词组命名，每个单词的首字母大写。
- 尽量避免大写，除非该缩写是众所周知的，比如 `URL`、`HTML` 等。
- 接口命名规则与类一样采用大驼峰命名法，多以`able` 或 `ible` 结尾，如`interface Runnable`、`interface Accessible` 。
- 若项目采用 `MVP` 架构，接口都以 `I` 为前缀，不加后缀，其他的接口采用上述命名规则。比如 `interface IUserTest`。
- 常见类名命名规则如下：

| 类                   | 描述                     | 举例                          |
| ------------------- | ---------------------- | --------------------------- |
| Activity 类          | 以 Activity 为后缀         | 登录页面类 LoginActivity         |
| Fragment 类          | 以 Fragment 为后缀         | 新闻标题列表 NewsTitlelFragment   |
| Service 类           | 以 Service 为后缀          | 下载服务 DownloadService        |
| Adapter 类           | 以 Adapter 为后缀          | 新闻详情适配器 NewsDetailAdapter   |
| 工具方法类               | 以 Utils 或 Manager 为后缀  | 日志工具 LogUtils               |
| 数据库类                | 以 DBHelper 为后缀         | 新闻数据库 NewsDBHelper          |
| 解析类                 | 以 Parser 为后缀           | JSON 解析类 JsonParser         |
| BroadcastReceiver 类 | 以 Receiver 为后缀         | 强制下线广播 ForceOfflineReceiver |
| 自定义共享基础类            | 以 Base 为前缀             | BaseActivity、BaseFragment   |
| 测试类                 | 以它要测试的类的名称开始，以 Test 结束 | UserTest                    |
| 抽象类                 | 以 Abstract 或 Abs 为前缀   | AbsUserTest                 |

### 1.2 变量命名

- 成员变量 / 局部变量
  - 使用小驼峰命名。
  - 不推荐使用谷歌的前面加 `m` 的编码风格。
- 控件变量
  - 使用小驼峰命名。
  - 建议使用 `控件缩写 + 逻辑名称` 格式，例如 `btnLogin`、`etUserName` 。
  - 对应的控件 `id` 命名为 `控件缩写_逻辑名称`，例如 `btn_login`、`et_user_name` 。
  - 常见控件缩写如下：

| 控件              | 缩写   | 控件           | 缩写   |
| --------------- | ---- | ------------ | ---- |
| TextView        | tv   | ImageButton  | ib   |
| EditText        | et   | CheckBox     | cb   |
| WebView         | wv   | RadioButton  | rb   |
| ImageView       | iv   | SeekBar      | sb   |
| VideoView       | vv   | ProgressBar  | pb   |
| MediaController | mc   | Spinner      | spr  |
| ListView        | lv   | SerachView   | sev  |
| GridView        | gv   | Button       | btn  |
| Gallery         | gly  | TimePicker   | tp   |
| LinearLayout    | ll   | DatePicker   | dp   |
| RelativeLayout  | rl   | ScrollView   | sv   |
| FrameLayout     | fl   | RecyclerView | rv   |

### 1.3 常量命名

- 单词每个字母均大写。
- 单词之间用下划线连接，力求语义表达完整清楚，不要嫌名字长。

### 1.4 方法命名

- 使用小驼峰命名。
- 方法名通常是动词或动词短语，保证见名知义，尽量不适用 `or` 或者 `and`，遵循 “ do one thing ” 原则。
- 一个方法尽量不要超过 50 行，如果方法太长，说明当前方法业务逻辑已经非常复杂，那么就需要进行方法拆分。

| 方法                       | 说明         | 方法                     | 说明         |
| ------------------------ | ---------- | ---------------------- | ---------- |
| initXX()                 | 初始化相关方法    | resetXX()              | 重置数据       |
| onXX()                   | 回调方法       | clearXX()              | 清除数据       |
| getXX()                  | 具有返回值的获取方法 | removeXX()             | 移除数据或视图    |
| setXX()                  | 设置方法       | drawXX()               | 绘制         |
| isXX()/hasXX()/checkXX() | 布尔型的判断方法   | displayXX()/showXX()   | 展示、提示信息的方法 |
| updateXX()               | 更新数据       | handleXX()/processXX() | 对数据进行处理的方法 |
| saveXX()                 | 保存数据       |                        |            |

### 1.5 资源文件命名

全部小写，采用下划线命名法。

#### 1.5.1 布局文件命名（xml 文件）

以对应的类别名称为前缀，逻辑名称在后，以下划线连接。

| 布局类型                 | 布局前缀       |
| -------------------- | ---------- |
| Activity             | activity_  |
| Fragment             | fragment_  |
| Include              | include_   |
| Dialog               | dialog_    |
| PopupWindow          | popup_     |
| Menu                 | menu_      |
| GridView 的item 布局文件  | item_grid_ |
| ListView 的 item 布局文件 | item_list_ |

#### 1.5.2 drawable 文件命名

以用途缩写作为前缀，逻辑名称在后，以下划线连接，区分状态时，添加状态后缀。

| drawable      | 规则         |
| ------------- | ---------- |
| 图标资源          | ic_        |
| 背景图片          | bg_        |
| 按钮图片          | btn_       |
| 分隔线           | div_       |
| 默认类           | def_       |
| 区分状态时，默认状态    | _normal    |
| 区分状态时，按下时的状态  | _pressed   |
| 区分状态时，选中时的状态  | _selected  |
| 区分状态时，不可用时的状态 | _disable   |
| 区分状态时，悬停效果    | _hovered   |
| 区分状态时，可选状态    | _checkable |

#### 1.5.3 动画文件命名（anim 文件夹下）

规则：动画类型_动画方向。

| 名称                | 说明      |
| ----------------- | ------- |
| fade_in           | 淡入      |
| fade_out          | 淡出      |
| push_down_in      | 从下方推入   |
| push_down_out     | 从下方推出   |
| push_left         | 推向左方    |
| slide_in_from_top | 从头部滑动进入 |
| zoom_enter        | 变形进入    |
| slide_in          | 滑动进入    |
| shrink_to_middle  | 中间缩写    |

#### 1.5.4 values 中 name 命名

##### 1.5.4.1 strings.xml

- 命名格式：xx 界面 + 逻辑功能 ，如 `activity_home_welcome_str`。
- 建议把同一个界面的所有 String 都在一起，方便查找。

##### 1.5.4.2 styles.xml

- 使用大驼峰命名法，主题可以命名为`XXTheme` ，控件的风格可以命名为 `XXStyle` 。

##### 1.5.4.3 colors.xml

- 命名格式：color_16进制颜色值


- 不要为某个控件指定特定颜色，比如 `bg_login` ，这样非常容易重复定义颜色值。

```xml
<resources>
  
    <color name="red_FF432F">#FF432F</color>
    <color name="red_FF0000">#FF0000</color>
  
</resources>
```

##### 1.5.4.4 dimens.xml

- 可以和 colors.xml 中命名格式类似
- 必要时也可用”逻辑名称_功能“ 命名

```xml
<resources>

    <!-- font sizes -->
    <dimen name="font_22">22sp</dimen>
    <dimen name="font_12">12sp</dimen>

    <!-- typical spacing between two views -->
    <dimen name="spacing_40">40dp</dimen>
    <dimen name="spacing_4">4dp</dimen>

    <!-- typical sizes of views -->
    <dimen name="button_height_60">60dp</dimen>
    <dimen name="button_height_40">40dp</dimen>

</resources>
```

## 2 注释规范

类、类属性、类方法的注释必须使用 Javadoc 规范，使用`/** XXX */ `格式，不得使用 `// XXX` 方式。

### 2.1 类和接口注释

类和接口统一添加 Javadoc 注释，要求至少写出创建者、创建时间以及内容简要说明。具体可以在 AS 中自己配制，`Settings → Editor → File and Code Templates → Includes → File Header`，格式如下：

```java
/**
 * author : xxx
 * e-mail : xxx@xx
 * time   : 2017/08/28
 * desc   :
 */
```

### 2.2 方法注释

下面几种方法，都必须添加 Javadoc 注释，除了返回值、参数、异常说明外，还必须指出该方法做什么事情，实现什么功能。

- 接口中定义的所有方法
- 抽象类中自定义的抽象方法
- 抽象父类的自定义公用方法
- 工具类的公用方法

```java
/** 方法的一句话概述
* 方法详述（简单方法可不必详述）
* @param s 说明参数含义
* @return 说明返回值含义
* @throws IOException 说明发生此异常的条件
* @throws NullPointerException 说明发生此异常的条件
*/
```

### 2.3 变量和常量注释

下面几种情况下的常量和变量，都要添加注释说明，优先采用右侧 `//` 来注释，若注释说明太长则在上方添加注释。

- 接口中定义的所有常量
- 公有类的公有常量
- 枚举类定义的所有枚举常量
- 实体类的所有属性变量

### 2.4 方法体内代码的注释

- 方法内部单行注释，在被注释语句上方另起一行，使用 `//` 注释。
- 方法内部多行注释使用 `/* ... */` 注释。


- 注意与代码对齐，`*` 及 `//` 与其后文字之间空一格。
- 不要在方法内部使用 Javadoc 形式的注释。

### 2.5 其他一些注释

- 资源文件代码注释

```xml
<!-- 注释内容 -->
```

- AS 已帮你集成了一些注释模板，我们只需要直接使用即可，在代码中输入`todo`、`fixme`等这些注释模板，回车后便会出现如下注释：

```java
// TODO: 2017/8/28 需要实现，但目前还未实现的功能的说明
// FIXME: 2017/8/28 需要修正，甚至代码是错误的，不能工作，需要修复的说明
```

## 3 格式规范

### 3.1 使用标准的大括号风格

- 大括号与`if, else, for, do, while`语句一起使用，即使只有一条语句(或是空)，也应该把大括号写上。
- 对于非空块和块状结构，大括号遵循 Kernighan 和 Ritchie 风格（R & N）
  - 左大括号前不换行
  - 右大括号前换行
  - 如果右大括号是一个语句、函数体或类的终止，则右大括号后换行；否则不换行。例如，如果右大括号后面是 `else` 或 `catch`，则不换行。

```java
class MyClass {
    public void method() {
      if (condition()) {
        try {
          something();
        } catch (ProblemException e) {
          recover();
        }
      }
    }
}
```

- 一个空的块状结构里什么也不包含，大括号可以简洁地写成`{}`，不需要换行。

### 3.2 限制代码行的长度

- 每行代码的长度不应该不超过 100 个字符。
- 例外：`package`和`import`语句，不换行。
- 例外：注释中包含了超过 100 个字符的命令示例或者 URL，为了便于剪切和复制，其长度可以超过 100 个字符。

### 3.3 合理空白

- 垂直空白
  - 方法体内的执行语句组、变量的定义语句组、不同的业务逻辑之间或者不同的语义之间插入一个空行。相同业务逻辑和语义之间不需要插入空行。
- 水平空白
  - 左小括号和右小括号与字符之间不出现空格。
  - `if/for/while/switch/do` 等保留字与括号之间都必须加空格。
  - 任何二目、三目运算符的左右两边都需要加一个空格。
  - 方法参数在定义和传入时，多个参数逗号后边必须加空格。
  - 请使用快捷键 `ctrl + alt + L` 格式化代码。

## 4 参考资料

- [Google Java编程风格指南](http://www.hawstein.com/posts/google-java-style.html)
- [阿里巴巴Java开发手册v1.2.0](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=100001199&idx=1&sn=c47917de3a4acc2b15186149ee918b82&chksm=692938a05e5eb1b6e1e075a2012cd4e17f09e3b175da682c8491221c73627cb52bbcc6f99d14&scene=20#rd)
- [Android 编码规范](http://jaeger.itscoder.com/android/2015/11/02/code-style-guideline-for-android.html)
- [Android 开发规范](https://github.com/Blankj/AndroidStandardDevelop#4-%E8%B5%84%E6%BA%90%E6%96%87%E4%BB%B6%E8%A7%84%E8%8C%83)
- [Android 开发规范](http://xhrong.github.io/2017/01/12/Android%E5%BC%80%E5%8F%91%E8%A7%84%E8%8C%83/)