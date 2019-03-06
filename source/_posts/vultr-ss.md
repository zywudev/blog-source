---
title: 科学上网 Vultr 搭建 SS 方法
date: 2019-03-06 20:03:27
tags: 科学上网
cover_img: https://raw.githubusercontent.com/zywudev/blog-source/master/image/img1.jpg
feature_img:
description:
keywords:
---

前几天，搬瓦工服务器到期了。准备再买的，但是最近搬瓦工被封的比较频繁，加上价格越来越高，想换一家试试，所以就找到了 vultr。

这里备忘一些 ss 连接服务器的命令。注册账户、购买、部署服务器步骤省略。

点击 [官网地址](https://www.vultr.com/?ref=7921104) 。

window 下需安装 [git](https://git-scm.com/)。

git 命令行输入：

``` shell
ssh root@45.76.111.96
```

接着输入密码（服务器密码，图中红框）。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/vultr1.png)

```shell
wget –no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

chmod +x shadowsocks.sh

./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

 以上三条命令输入完成后会提示输入密码，这个密码是自己设置登录 ss 的密码。

然后输入端口号、选择加密方式。

按任意键确认，出现以下信息表示搭建成功。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/vultr2.png)

下载 [ss 客户端](https://github.com/shadowsocks)连接即可。

