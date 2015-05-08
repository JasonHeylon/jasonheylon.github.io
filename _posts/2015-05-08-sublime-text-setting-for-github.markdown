---
layout: post
title:  "sublime text 设置导致代码push到github后tab显示不正确"
date:   2015-05-08 12:00:00
categories: ruby
comments: true
---

在sublime中默认是不会把tab转换成空格(blankspace)的, 这样的代码push到github后，如果github web页面查看代码将会让人抓狂的。

所以在sublime中一定要写下如下设置， 一定要

{% highlight ruby %}
  // Set to true to insert spaces when tab is pressed
  "translate_tabs_to_spaces": true,
{% endhighlight %}
