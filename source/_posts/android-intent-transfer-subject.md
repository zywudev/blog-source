---
title: Android Intent 传递自定义对象
date: 2017-08-17 16:03:59
tags: Android
toc: true
---

Intent 可以用来启动活动、发送广播、启动服务等，通过 `putExtra` 方法可以添加一些附加数据，达到传值的效果，但若想传递自定义对象的时候就无能为力了。

可以通过使用 Serializable 接口、Parcelable 接口以及转换对象为字符串的方式进行传递。

**1、Serializable**

表示将一个对象转为字节实现可存储或可传输的状态，一个对象能够序列化的前提是实现 Serializable 接口。

Model：

```java
public class Person implements Serializable {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

传递对象：

```java
Person person = new Person();
person.setName("wuzy");
Intent intent = new Intent(MainActivity.this, Main2Activity.class);
intent.putExtra("person_data",person);
startActivity(intent);
```

接收对象：

```java
Person person = (Person) getIntent().getSerializableExtra("person_data");
```

**2、Parcelable**

将一个完整的对象进行分解，分解后的每一部分都是 Intent 所支持的数据类型。

Model：

```java
public class Person implements Parcelable {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    /*
    将想要传递的数据写入到 Parcel 容器中
     */
    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeString(name);
    }

    public static final Creator<Person> CREATOR = new Creator<Person>() {
        /*
        用于将写入 Parcel 容器中的数据读出来，用读出来的数据实例化一个对象，并且返回
         */
        @Override
        public Person createFromParcel(Parcel in) {
            Person person = new Person();
            person.setName(in.readString());
            return person;
        }

        /*
        创建一个长度为 size 的数组并且返回,供外部类反序列化本类数组使用
         */
        @Override
        public Person[] newArray(int size) {
            return new Person[size];
        }
    };
}
```

传递对象：

```java
Person person = new Person();
person.setName("wuzy");
Intent intent = new Intent(MainActivity.this, Main2Activity.class);
intent.putExtra("person_data",person);
startActivity(intent);
```

接收对象：

```java
Person person =  getIntent().getParcelableExtra("person_data");
```

**3、转化为 JSON 字符串**

Model：

```java
public class Person {
    private String name;
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

传递对象：

```java
Person person = new Person();
person.setName("wuzy");
Intent intent = new Intent(MainActivity.this, Main2Activity.class);
intent.putExtra("person_data",new Gson().toJson(person));
startActivity(intent);
```

接受对象：

```java
String personJson = getIntent().getStringExtra("person_data");
Person person =  new Gson().fromJson(personJson, Person.class);
```

**速度上，使用Parcelable 速度最快，Serializable 次之，转换为 JSON 字符串最慢。**

