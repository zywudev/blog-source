---
title: 自动更新 hosts 实现科学上网
date: 2017-07-05 09:38:24
tag: 科学上网
toc: true
---

众所周知，在国内上 Google 查阅资料非常困难。于是出现了一些免费的工具和 VPN 服务帮我们访问，但要么是有流量限制，要么需要收费，而且有些 VPN 代理的速度也不是很快。

更改 hosts 方法实现科学上网是最简单可行的方式，但 hosts 会经常被封，每次手动更新很麻烦。所以，下面我介绍一个自动更新 hosts 的小工具，可以很方便的访问 Google。

1、首先安装 SwitchHosts，SwitchHosts 是一个用于快速切换 hosts 文件的小程序，非常实用，其强大之处在于它可以自动关联远程 hosts，自动更新。下载地址：[https://oldj.github.io/SwitchHosts/](https://oldj.github.io/SwitchHosts/)

2、安装完成后，点击软件界面左下角的「+」号按钮，创建一个新的 hosts 分组，在弹出的界面中，选择「Remote」类型，「Hosts title」随意填写，姑且命名 「科学上网」。「URL」中填入 [https://raw.githubusercontent.com/racaljk/hosts/master/hosts](https://raw.githubusercontent.com/racaljk/hosts/master/hosts)，该 hosts 是 GitHub 上的一个开源项目，会不定期更新可用的 hosts 文件，项目地址：[https://github.com/racaljk/hosts](https://github.com/racaljk/hosts)。「Auto refresh」设置更新 hosts 时间周期，姑且选择 24小时。点击「OK」按钮即可。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/SwitchHosts.jpg)

3、最后打开「科学上网」右侧的开关按钮，就可以无忧无虑的科学上网了。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/SwitchHost2.jpg)

PS：需要注意的是这种方法只能访问 YouTube 网站，但是不能播放视频。主要原因由于 Youtube 的 SN 编码域名（[编码规则](https://github.com/lennylxx/ipv6-hosts/wiki/YouTube#4-sn-%E7%BC%96%E7%A0%81%E5%9C%B0%E5%9D%80)）为固定 IP，并且这些 IP 基本已被封锁殆尽，因此即使能打开 Youtube 首页，也无法加载其中的视频。其解决方法可以访问[在线视频转码网站](https://www.onlinevideoconverter.com/video-converter)，下载相关 YouTube 视频。



