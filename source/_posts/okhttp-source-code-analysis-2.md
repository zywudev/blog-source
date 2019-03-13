---
title: OkHttp 源码分析（二）—— 缓存机制
date: 2019-03-11 16:24:08
tags:
- Android
- 源码分析
cover_img:
feature_img:
description:
keywords:
---

[上一篇文章](http://wuzhangyang.com/2019/03/11/okhttp-source-code-analysis-1/)我们主要介绍了 OkHttp 的请求流程，这篇文章讲解一下 OkHttp 的缓存机制。

在讲解 OkHttp 的缓存机制之前，先了解下 Http  的缓存理论知识，这是实现 OKHttp 缓存的基础。

## Http 缓存

### 缓存存储策略

缓存存储策略的作用就是决定 Http 响应内容是否可以缓存到客户端。

Http/1.1 定义的

### 缓存过期策略

### 缓存对比策略

