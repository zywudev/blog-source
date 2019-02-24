---
title: GitHub Pages + Hexo 搭建博客
date: 2017-03-04 17:53:39
toc: true
tag: Hexo
---
第一篇博客，记录一下博客搭建过程。

## 安装 node.js
[node.js](https://nodejs.org/en/)

## 安装 Git
[Git](https://git-scm.com/downloads)

## 安装 Hexo 
在文件夹中建立名为 `hexo` 的文件夹，右键打开 Git Bush，使用 npm 安装 Hexo。

```java
npm install hexo-cli -g
```

初始化 blog， Hexo 自动在 blog 文件夹下创建网站所需文件。

```java
hexo init blog
```

进入 blog 文件夹，安装依赖包。

```java
cd blog
npm install
```

生成静态页面

```java
hexo g # 或 hexo generate 
```

启动本地 web 服务

```java
hexo s # 或 hexo server
```

此时在浏览器地址栏中键入 [http://localhost:4000/](http://localhost:4000/) ， 可以看到内置的页面。

![image](https://raw.githubusercontent.com/zywudev/blog-source/master/image/hexo-init-page.png)
## GitHub Pages 设置
注册 GitHub 及其使用可以参考 [从 0 开始学习 GITHUB 系列汇总](http://stormzhang.com/github/2016/06/19/learn-github-from-zero-summary/)。

在 GitHub 上创建仓库，而且仓库的名字格式为： `username.github.io`，username 与 GitHub 账号名对应，每个帐号只能有一个仓库来存放个人主页。可以通过 [http://username.github.io](http://username.github.io) 来访问个人主页。
## 部署 Hexo 到 GitHub Pages
其实就是将 Hexo 生成的静态页面提交 (git commit) 到 GitHub 上。

在 Hexo 根目录下的配置文件 `_config.yml` 中进行修改:

```java
deploy:
  type: git
  repo: git@github.com:zywudev/zywudev.github.io.git
  branch: master
```

还得安装一个扩展

```java
npm install hexo-deployer-git --save
```

然后在命令行部署

```java
hexo d
```

## Hexo 主题基本配置
选择喜欢的主题。Hexo 官网[主题](https://hexo.io/themes/)。我的博客选用的是 NexT 主题，官方提供了详细的[使用文档](http://theme-next.iissnan.com/)。

### 下载 NexT 主题

```java
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 启用 NexT 主题
克隆完成后，打开站点配置文件 `__config.yml`，找到 `theme` 字段，修改为：

```java
theme: next
```

> 站点配置文件：博客目录下的 `__config.yml` 文件。

> 主题配置文件： `themes/next` 目录下的 `__config.yml` 文件。

### 启动本地 web 服务验证

```java
hexo s --debug
```

至此，即完成了基于 GitHub Pages + Hexo 的个人博客框架搭建。

## 博客推广
将个人博客推广到 Google 搜索引擎上。

### 验证网站
推荐文件验证。

下载 HTML 验证文件，将该文件放到博客 source 目录下。

hexo 编译文件时，会给下载的 HTML 文件中添加其他的内容，导致验证失败，所以需要在文件开头添加 layout: false 来取消 hexo 对其进行的转换。

```java
layout: false   ---
google-site-verification: googleb6fc53a32f5418d9.html
```

部署到 GitHub，输入[https://zywudev.github.io/googleb6fc53a32f5418d9.html](https://zywudev.github.io/googleb6fc53a32f5418d9.html) ， 能访问即可点击验证按钮进行验证。

### 站点地图
> 站点地图是一种文件，您可以通过该文件列出您网站上的网页，从而将您网站内容的组织架构告知 Google 和其他搜索引擎。Google 等搜索引擎网页抓取工具会读取此文件，以便更加智能地抓取您的网站。

使用 hexo-generator-sitemap 插件来生成 Sitemap，执行

```java
npm install hexo-generator-sitemap --save
```

执行

```java
hexo g
```

博客根目录的public下面生成了 `sitemap.xml` 。如果没有，在博客目录的 `_config.yml` 中添加如下代码重新编译

```java
sitemap:
path: sitemap.xml
```

要将博客目录 `__config.yml` 中的 url 字段设置为自己的域名

```java
url: http://zywudev.github.io
```

部署到 Github，访问 [https://zywudev.github.io/sitemap.xml](https://zywudev.github.io/sitemap.xml)。

提交 sitemap 到 [Google站长工具](https://www.google.com/webmasters/tools/home?hl=zh-CN)，找到添加站点地图按钮，添加 `sitemap.xml` 即可，如下图。

![](https://raw.githubusercontent.com/zywudev/blog-source/master/image/google-sitemap.png)


## 博客优化
### 添加 about 页面
```java
hexo new page "about" 
```

在 `\source\about\` 目录下会生成一个 `index.md` 文件,添加个人信息即可。
### 添加分类、标签页面

```java
hexo new page "categories"
hexo new page "tags" 
```

### 博客标题、作者等
打开站点配置文件
```java
title: Wu's blog 
subtitle:
description: O ever youthful,O ever weeping
author: Wu 
language: zh-Hans
timezone:
```
### 修改文章内文本链接样式
将文本链接设置为蓝色，修改 
`themes\next\source\css\_common\components\post\post.styl`,
添加

```java
.post-body p a{
  color: #0593d3;
  border-bottom: none;

  &:hover {
    color: #0477ab;
    text-decoration: underline;
  }
}
```

### 修改博客 logo
通过网站 [favicon在线制作](http://tool.lu/favicon/) 网站制作 favicon 图片。

将图片放在博客 `source` 目录下即可。

### 添加 Fork me on GitHub 挂件

在[官网](https://github.com/blog/273-github-ribbons)选取样式。

拷贝代码，修改 href 地址为自己的 GitHub 地址
```java
<a href="https://github.com/you">
改为：
<a href="https://github.com/zywudev">
```

修改文件: `themes/next/layout/_layout.swig`，将代码拷贝至对应位置。

```java
<body>

  {% include '_scripts/third-party/analytics.swig' %}

  <div class="{{ container_class }} {% block page_class %}{% endblock %} ">
    <div class="headband"></div>
    
    <!--------------add Fork me on GitHub ------------->
    
    <a href="https://github.com/zywudev"><img style="position: absolute; top: 0; left: 0; border: 0;" src="https://camo.githubusercontent.com/8b6b8ccc6da3aa5722903da7b58eb5ab1081adee/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f6c6566745f6f72616e67655f6666373630302e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_left_orange_ff7600.png"></a>

	<!--------------add Fork me on GitHub ------------->
		
    <header id="header" class="header" itemscope itemtype="http://schema.org/WPHeader">
      <div class="header-inner"> {%- include '_partials/header.swig' %} </div>
    </header>
```
## 参考链接

[如何搭建一个独立博客——简明Github Pages与Hexo教程](http://www.jianshu.com/p/05289a4bc8b2)
[手把手教你使用Hexo + Github Pages搭建个人独立博客](https://linghucong.js.org/2016/04/15/2016-04-15-hexo-github-pages-blog/)
[博客推广——提交搜索引擎](http://selfboot.cn/2014/12/21/add_blog_to_google/#站点地图)

（完）
