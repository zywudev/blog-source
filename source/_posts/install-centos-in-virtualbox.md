---
title: 在 VirtualBox 中安装 CentOS 7
date: 2017-06-17 22:33:53
---

本文主要介绍在 VirtualBox 虚拟机中安装 CentOS 7 系统的方法和步骤。

## 一、准备

1、在 VM VirtualBox 的[官网](https://www.virtualbox.org/)下载安装虚拟机， VirtualBox 是 Oracle 下一款免费的虚拟机软件。

2、下载 CentOS 系统，有以下站点可以下载。

- [国家高速网络中心](http://ftp.twaren.net/Linux/CentOS/7/isos/)
- [昆山科技大学](http://ftp.ksu.edu.tw/FTP/Linux/CentOS/7/isos/)
- [CentOS 官方网站](http://mirror.centos.org/centos/7/isos/)

在不同站点中下载的文件名会是 CentOS-7-x86_64-DVD-1611.iso 这样的格式，其中 CentOS 表示 7.x 版本，x86_64 指的是 64 位操作系统，1611 指的是 2016 年的 11 月发布的版本，直接下载 DVD 版本的镜像文件。

## 二、创建虚拟电脑

1、 打开 VirtualBox 虚拟机，在主界面上点击“**新建**”，在名称中输入 centos ,“**类型**” 和“**版本**”会自动更新。点击“**下一步**”。

![](http://om9o63aks.bkt.clouddn.com/centos-1.jpg)

2、内存大小一般不超过主机内存的一半，这里我设置 CentOS 系统内存为 1G。点击“**下一步**”。

![](http://om9o63aks.bkt.clouddn.com/centos-2.jpg)

3、点选“现在创建虚拟硬盘”（这个是默认选中的），然后单击“**创建**”。

![](http://om9o63aks.bkt.clouddn.com/centos-3.jpg)

4、使用默认的“VDI” （VirtualBox 磁盘映像）。然后单击“**下一步**”。

![](http://om9o63aks.bkt.clouddn.com/centos-4.jpg)

5、选择“**动态分配**”。动态分配是随着使用慢慢增大占用实际物理硬盘的空间，最大到分配的大小；固定则是一次性分配好空间。单击“**下一步**”。

![](http://om9o63aks.bkt.clouddn.com/centos-5.jpg)

6、文件位置就是虚拟机创建后存放的位置，windows 默认放在系统盘，如果系统盘空间足够，默认就好。要么点击右边的文件夹图标可以更换位置。我给 CentOS 分配 20G 的硬盘空间。点击“**创建**”。

![](http://om9o63aks.bkt.clouddn.com/centos-6.jpg)

7、现在虚拟电脑就创建好了，接下来就是给电脑装 CentOS 系统了。

![](http://om9o63aks.bkt.clouddn.com/centos-7.jpg)

## 三、给虚拟电脑安装 CentOS 系统

1、在 VirtualBox 主界面上选中 CentOS 系统，点击“**设置**”，然后点选择 “系统”，将光驱升到第一位，这样可以读取下载好的 CentOS 系统镜像文件。

![](http://om9o63aks.bkt.clouddn.com/centos-8.jpg)

2、选中“**存储**”，单击没有碟片，点击右边的光盘图标，选择提前下载好的 CentOS-7 系统镜像打开。点击"**OK**"。

![](http://om9o63aks.bkt.clouddn.com/centos-9.jpg)

3、点击 VirtualBox 主界面的“**启动**”，进入安装系统界面，选择 “**Install CentOS Linux**”，回车。 

![](http://om9o63aks.bkt.clouddn.com/centos-10.jpg)

4、进入设置界面。

![](http://om9o63aks.bkt.clouddn.com/centos-11.jpg)

5、对 Linux 比较熟悉的，软件选择可以选择“最小安装”，这是纯命令行模式。对于像我一样刚刚接触 Linux 的可以选择带有桌面环境的 GNOME 或 KDE Plasma Workspaces 环境。![](http://om9o63aks.bkt.clouddn.com/centos-13.jpg)

6、设置 root 密码，这是超级管理员，权限最高。最好创建一个一般账号用于登录，特殊需要时才使用 root 管理员登录。点击“**开始安装**”。

![](http://om9o63aks.bkt.clouddn.com/centos-12.jpg)

然后就静静等待。安装完成后，启动系统，输入账号和密码，开始体验 CentOS 系统吧。




