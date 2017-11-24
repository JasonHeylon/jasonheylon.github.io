---
layout: post
title:  "sublime text 设置导致代码push到github后tab显示不正确"
date:   2015-05-08 12:00:00
categories: ruby
comments: true
---

在sublime中默认是不会把tab转换成空格(blankspace)的, 这样的代码push到github后排版很有问题。
所以要在subline里设置

{% highlight ruby %}
  // Set to true to insert spaces when tab is pressed
  "translate_tabs_to_spaces": true,
{% endhighlight %}
