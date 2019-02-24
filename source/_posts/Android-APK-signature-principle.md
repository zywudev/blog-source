---
title: Android APK 签名原理
date: 2018-07-10 20:46:08
tags: Android
toc: true
---

Android APK 签名原理涉及到密码学的加密算法、数字签名、数字证书等基础知识，这里做个总结记录。

### 非对称加密

需要两个密钥，一个是公开密钥，另一个是私有密钥；一个用作加密的时候，另一个则用作解密。

其相对的加密即为对称加密，可以用现实世界中的例子来对比：一个传统保管箱，开门和关门都是使用同一条钥匙，这是对称加密；而一个公开的邮箱，投递口是任何人都可以寄信进去的，这可视为公钥；而只有邮箱主人拥有钥匙可以打开邮箱，这就视为私钥。

### 消息摘要算法

一种能产生特殊输出格式的算法，其原理是根据一定的运算规则对原始数据进行某种形式的信息提取，被提取出的信息就被称作原始数据的消息摘要。著名的摘要算法有 RSA 公司的 MD5 算法和 SHA-1 算法及其大量的变体。

消息摘要算法的主要特点：

- 变长输入，定长输出。即不管输入多长，输出永远是相同的长度。
- 输入不同，输出不同。输入相同，输出相同。
- 单向、不可逆。即只能进行正向的消息摘要，而无法从摘要中恢复出任何的原始消息 。

消息摘要的作用：保证了消息的完整性。如果发送者发送的信息在传递过程中被篡改，那么接受者收到信息后，用同样的摘要算法计算其摘要，如果新摘要与发送者原始摘要不同，那么接收者就知道消息被篡改了。

### 数字签名

数字签名是对非对称加密和消息摘要技术的具体应用。其目的就是确保消息来源的可靠性。

消息发送者生成一对公私钥对，将公钥给消息的接收者。如果发送者要发送信息给接收者，会进行如下三步操作：

- 通过消息摘要算法提取消息摘要。
- 使用私钥，对这个摘要加密，生成数字签名。
- 将原始信息和数字签名一并发给接收者。

接收者在收到信息后通过如下两步验证消息来源的真伪。

- 使用公钥对数字签名进行解密，得到消息的摘要，由此可以确定信息是又发送者发来的。
- 对原始信息提取消息摘要，与解密得到的摘要对比，如果一致，说明消息在传递的过程中没有被篡改。

以上的数字签名方法的前提是，接收者必须要事先得到正确的公钥。如果一开始公钥就被别人篡改了，那坏人就会被你当成好人，而真正的消息发送者给你发的消息会被你视作无效的。如何保证公钥的安全性？这就需要数字证书来解决。

### 数字证书

发送者的公钥的安全合法性需要一个公钥做认证，而这个公钥的合法性又该如何保证？这个问题可以无限循环下去，无法到头了。所以需要一个可信的机构来提供公钥，这种机构称为认证机构 (Certification Authority， CA)。CA 就是能够认定”公钥确实属于此人”，并能生成公钥的数字签名的组织或机构。 

CA 用自己的私钥，对发送者的公钥和一些相关信息一起加密，生成"数字证书"。发送者在签名的时候，带上数字证书发送给接收者。接收者用 CA 的公钥解开数字证书，就可以拿到发送者真实的公钥了，然后就能证明"数字签名"是否来源真实。

对于数字签名和数字证书，可以查看阮一峰老师的文章《 [数字签名是什么？](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)》，讲得很简单易懂。

### Android APK 签名流程

为了防止 APK 在传送的过程中被第三方篡改，Google 引入了签名机制。

签过名的 APK 文件比未签名的 APK 文件多了一个 META-AF 文件夹，包含以下三个文件。签名的信息就在这三个文件中。

```java
MANIFEST.MF
CERT.RSA
CERT.SF
```

APK 的签名主要有以下几个流程：

1、对 APK 文件夹中的文件逐一遍历进行 SHA1 （或者 SHA256）算法计算文件的消息摘要，然后进行 BASE64 编码后，作为 “SHA1-Digest” 属性的值写入到 MANIFEST.MF 文件中的一个块中。该块有一个 “Name” 属性，其值就是该文件在 APK 包中的路径。

```java
Manifest-Version: 1.0
Built-By: Generated-by-ADT
Created-By: Android Gradle 3.1.0

Name: AndroidManifest.xml
SHA1-Digest: 9hTSmRfzHEeQc7V2wxBbTT3DmCY=

Name: META-INF/android.arch.core_runtime.version
SHA1-Digest: BeF7ZGqBckDCBhhvlPj0xwl01dw=

Name: META-INF/android.arch.lifecycle_livedata-core.version
SHA1-Digest: BeF7ZGqBckDCBhhvlPj0xwl01dw=
```

2、计算这个 MANIFEST.MF 文件的整体 SHA1 值，再经过 BASE64 编码后，记录在 CERT.SF 主属性块（在文件头上）的 “SHA1-Digest-Manifest” 属性值值下。然后，再逐条计算 MANIFEST.MF 文件中每一个块的 SHA1，并经过 BASE64 编码后，记录在 CERT.SF 中的同名块中，属性的名字是 “SHA1-Digest” 。

```java
Signature-Version: 1.0
Created-By: 1.0 (Android)
SHA1-Digest-Manifest: MJQyZ0dc4dv7G9nlJPAMQLwEwbU=
X-Android-APK-Signed: 2

Name: AndroidManifest.xml
SHA1-Digest: IJioMmfD693T4qnUJcPKhq9woHQ=

Name: META-INF/android.arch.core_runtime.version
SHA1-Digest: OPQCkzMXJVPQryHeMowVNZmfRMw=

Name: META-INF/android.arch.lifecycle_livedata-core.version
SHA1-Digest: TSBGEIW1zN2n2sraHWcuRYSO8JU=
```

3、把之前生成的 CERT.SF 文件， 用私钥计算出签名, 然后将签名以及包含公钥信息的数字证书一同写入  CERT.RSA  中保存。

```java
3082 02f9 0609 2a86 4886 f70d 0107 02a0
8202 ea30 8202 e602 0101 310b 3009 0605
2b0e 0302 1a05 0030 0b06 092a 8648 86f7
0d01 0701 a082 01e1 3082 01dd 3082 0146
0201 0130 0d06 092a 8648 86f7 0d01 0105
0500 3037 3116 3014 0603 5504 030c 0d41
6e64 726f 6964 2044 6562 7567 3110 300e
0603 5504 0a0c 0741 6e64 726f 6964 310b
3009 0603 5504 0613 0255 5330 1e17 0d31
```

### 签名校验

1、完整性校验

如果 APK 中文件被修改，对应 MANIFEST.MF 中的 SHA1 会发生改变。

2、APK 作者身份唯一性校验

当在 Android 设备上安装  APK 包时，会从存放在 CERT.RSA 中的公钥证书中提取公钥，进行 RSA 解密来校验安装包的身份。 使用不同的 key 生成的签名信息会不同，不同的私钥对应不同的公钥，因此最大的区别是签名证书中存放的公钥会不同，所以我们可以通过提取 CERT.RSA 中的公钥来检查安装包是否被重新签名了。 