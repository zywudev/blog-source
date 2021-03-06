---
title: Android 加固技术调研
date: 2018-10-09 19:09:25
tags: Android 加固
categories: Android
---

## 第一代加固

第一代加固原理是基于 Dex 加载器的加固技术。

### 基本步骤

1、从 Apk 文件中获取原始的 dex 文件。

2、对原始的 dex 文件进行加密，并将加密后的 dex 文件存放在 asset 目录。

3、用脱壳 dex 文件替换原始 apk 文件的 dex 文件。脱壳  dex 文件的作用主要有两个：一是解密加密后的 dex 文件，二是动态加载解密后的 dex 文件。

4、修改清单文件，将程序入口改为壳程序。

5、打包签名。

### 缺陷

依赖 Java 的动态加载机制，解密后的 dex 文件必须解压到文件系统，即使是应用的私有目录，攻击者很容易获取文件。

## 第二代加固--不落地加载

相对于第一代加固，第二代加固在加载原始 dex 文件时采用的是内存加载方案。

即在解密原始 dex 文件后，不需要将 dex 写入文件系统，系统直接读取 dex 字节进行加载。

Android 底层支持内存加载 dex，但是 java 层未实现内存夹杂的接口，可以通过 jni 层调用底层的内存加载 dex 的函数。

市面上绝大多数第三方加固厂商的加密方案都是基于第二代加固技术。

### 缺陷

第二代加固方案能够防止第一代加固技术文件必须落地易被复制的缺陷，但是解密后的 dex 文件加载到内存后，在内存中是连续的 ，利用 gdb 等调试工具 dump 内存后可以直接找到原始 dex。

## 第三代加固--指令抽离

由于第二代加固技术仅仅对文件级别进行加密，其带来的问题是内存中的 Payload 是连续的，可以被攻击者轻易获取。第三代加固技术对这部分进行了改进，将保护级别降到了函数级别。

### 基本步骤

1、打包阶段将 Dex 文件中要保护的核心函数抽离出来生成另外一个文件。

2、运行阶段将函数内容重新恢复到对应的函数体。恢复的时间点有如下几种方式。

- 加载之后恢复函数内容到 dex 壳所在的内存区域
- 加载之后将函数内容恢复到虚拟机内部的结构体上：虚拟机读取 dex 文件后内部对每一个函数有一个结构体，这个结构体上有一个指针指向函数内容，可以通过修改这个指针修改对应的函数内容。
- 拦截虚拟机与查找执行代码相关的函数，返回函数内容。

### 缺陷

攻击者可以通过自定义 Android 虚拟机，在解释器的代码上做记录一个函数的内容。接下来遍历所有函数，从而获取全部的函数内容。最终重组成一个完整的 dex 文件。

## 第四代加固--指令转换/VMP

第三代加固技术是函数级别的保护，使用 Android 虚拟机内的解释器执行代码，带来可能被记录的缺陷。

第四代加固技术使用自己的解释器来避免第三代的缺陷。

在编译打包的时候将 dex 的核心函数抽离，抽离后，翻译成一种自己定义的指令，用自己的一种编译指令进行翻译，把这个指令变一个种，变成其他的指令，这个时候运行的时候通过自己的解释器来解释执行，是自己定义的相关指令。在内存中运行的指令，在某些保护的函数里面就一定不是谷歌的标准指令了，这点能够很有效的防止内存直接拷贝等破解方案。

### 缺陷

其必须通过虚拟机提供的JNI接口与虚拟机进行交互，攻击者可以直接将指令转换/VMP 加固方案当作黑盒，通过自定义的 JNI 接口对象，对黑盒内部进行探测、记录和分析，进而得到完整 dex程序。

## 参考

[APP加固技术历程及未来级别方案：虚机源码保护](https://juejin.im/entry/5a16915e51882575d144a692)