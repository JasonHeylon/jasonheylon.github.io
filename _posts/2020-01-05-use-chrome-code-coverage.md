---
layout: post
title: "使用Chrome检测代码利用率(Code Coverage)"
date: 2020-01-05 00:00:00
categories: frontend
comments: true
---

# 什么是代码利用率(Code Coverage)

代码利用率对于前端来说就是在页面被加载展示后，所下载JavaScript和CSS是否被充分利用了，所下载的JavaScript中的代码是否都被执行了，执行的比例。CSS中的每一条样式是否都被渲染了，渲染的比例。

# 为什么需要了解Code Coverage

web应用的性能最重要的因素之一就是加载速度，而影响加载速度中最重要的因素就是所需要下载资源的大小，通常来说只加载必要的资源是优化前端性能重要的第一步。

# 如何知道Code Coverage

Chrome 提供了一个工具查看Code Coverage。

比如查看一下baidu首页的Code Coverage。
首先打开chrome 输入baidu.com并打开开发者工具，按`cmd+shift+p`会打开一个输入框，键入`coverage` 选择`Show Coverage`回车，之后就会看到coverage 面板。点击面板上的刷新按钮。数据如下


![baidu的CodeCoverage](/assets/posts/2020-01-05/baidu-coverage.jpg "baidu的Code Coverage")

点击其中的文件后还可以看到当前文件内具体哪些方法被使用，哪些没有
![baidu的CodeCoverage](/assets/posts/2020-01-05/baidu-coverage-file.jpg "baidu的Code Coverage")
