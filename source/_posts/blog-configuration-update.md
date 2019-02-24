---
title: 博客配置更新
date: 2017-04-19 13:51:29
tag: Hexo
toc: true
---

今天，将博客主题由 [next](https://github.com/iissnan/hexo-theme-next) 换成了  [scribble](https://github.com/saintwinkle/hexo-theme-scribble)，关于博客的安装和 next 主题的配置详见 [GitHub Pages + Hexo 搭建博客](http://wuzhangyang.com/2017/03/04/GitHub-Pages-Hexo-build-blog/)。 然后对博客添加了些许功能。

## 添加网易云跟帖
由于多说评论即将停用，所以添加了网易云跟帖作为博客的评论系统。
首先登录[网易云跟帖](https://gentie.163.com/index.html)，然后进入`后台管理`，填写`站点信息`，站点网址处需填自己申请的域名，填写 github.io 域名会提示站点名称或URL已经存在。 
![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/zhandianxinxi.png)
然后点击`获取代码`，复制通用代码，拷贝至需要放置评论的位置即可。这里要注意本地服务器预览不到效果，部署后，才能看到评论。
![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/huoqudaima.png)

## 绑定独立域名
### 购买域名
首先在[万网](https://wanwang.aliyun.com/)上购买域名，当然也可以去 [GoDaddy](https://sg.godaddy.com/zh/) 购买。

### 绑定域名与GitHub Pages

首先，修改域名的DNS地址为 `f1g1ns1.dnspod.net` 和 `f1g1ns2.dnspod.net` 。
![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/modifyDNS.png)
然后在本地站点目录里的 `source` 目录下添加一个 `CNAME` 文件，不带后缀,文件名要大写。里面添加域名信息（不加 http）。如：`wuzhangyang.com` 。执行 `hexo d -g` 部署到 GitHub 上。
注册 [DNSPod](https://www.dnspod.cn/)， 添加域名，添加记录。
![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/addnotes.png)

> 在记录中，`192.30.252.153` 与 `192.30.252.153` 是 Github Pages 服务器指定的 IP 地址。
记录类型为 CNAME，记录值是 `zywudev.github.io.` 。在记录值中.io后面还有一个小数点。

最后把更改 deploy 到 GitHub 上。
到此，域名绑定好了，在 GitHub 博客仓库的 Settings 里面可以看见以下提示。
![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/dns_success.png)