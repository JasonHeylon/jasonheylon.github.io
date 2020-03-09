---
layout: post
title: "CSS3 Flex"
date: 2019-12-30 00:00:00
categories: frontend
comments: true
---

# Flex

## 基本知识

[Flex Box Specification](https://drafts.csswg.org/css-flexbox/)

### FlexBox 相关属性

1. 创建FlexBox: `display`
2. 方向控制: `flex-flow` (`flex-direction`, `flex-wrap`的简写方式)
3. 排列控制: `justify-content`, `align-items`, `align-self`, `align-content`
4. 排序: `order`
5. 对变化的适应: `flex` (`flex-grow`, `flex-shrink`, `flex-basis`的简写方式)

### display属性

- `flex`: 创建一个块级的FlexBox容器
- `inline-flex`: 创建一个行内的FlexBox容器

### Flex Item

- 所有通过display属性创建的FlexBox容器元素 的一级后代元素都会成为Flex Item.
- 文本


```html
  <article>
    There is a <span>span</span> and a <a href="www.baidu.com">link</a> here
  </article>
```

上面代码如果添加了CSS `article { display: flex }`后, 将会生成以下5个Flex Item:

- `This is a`
- `<span>span</span>`
- `and a`
- `<a href="www.baidu.com">link</a>`
- `here`

### 非Flex Item
- `::first-line`, `::first-letter`
- 空白区域

### 受影响的CSS属性

以下属性将会被影响：
1. `margin` 相邻的Flex Item 将不再有margin距离的折叠
2. `min-width` 和 `min-height` 默认值为`auto` (原来默认为0)
3. `visibility` 默认值为 `collapse`

以下属性将会被忽略
1. `column-*` 系列属性
2. `float`
3. `clear`
4. `vertical-align`


## Flex容器的主要属性
 包括 `flex-direction`, `flex-wrap`, `flex-flow`, `justify-content`, `align-items`, `align-content`


### flex-direction属性

`flex-direction` 可以设置 Flex Item的排列方向默认值为`row`

- `row` 横向排列
- `row-reverse` 横向反向排列
- `column` 纵向排列
- `column-reverse` 纵向反向排列

#### 主轴 main-axis, 交叉轴cross-axis

`flex-direction`控制的是Flex主轴(main-axis)的方向, 如下图

![Flex main-axis](/assets/posts/2019-12-30-flex-and-grid/flex-main-axis.jpg "Flex main-axis")

- 当`flex-direction: row` 主轴的方向是 从左到右, 交叉轴为从上到下
- 当`flex-direction: row-reverse` 主轴的方向是 从右到左, 交叉轴为从上到下
- 当`flex-direction: column` 主轴的方向是 从上到下, 交叉轴为从左到右
- 当`flex-direction: row` 主轴的方向是 从下到上, 交叉轴为从左到右


### flex-wrap

`flex-wrap` 默认为 `nowrap`

- `nowrap` 不换行
- `wrap` 主轴的一行宽度超过容器宽度时进行换行, 换行导致的新增行将 依照交叉轴方向插入
- `wrap-reverse` 主轴的一行宽度超过容器宽度时进行换行, 换行导致的新增行将 依照交叉轴反方向插入

{% raw %}
<iframe height="302" style="width: 100%;" scrolling="no" title="zYxaQMV" src="https://codepen.io/jason-heylon/embed/zYxaQMV?height=302&theme-id=dark&default-tab=css,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/jason-heylon/pen/zYxaQMV'>zYxaQMV</a> by Jason  Heylon
  (<a href='https://codepen.io/jason-heylon'>@jason-heylon</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
{% endraw %}


### flex-flow

`flex-flow`是 `flex-direction`和`flex-wrap`的缩写形式， 如`flex-flow: row no-wrap`

### justify-content

`justify-content`属性为 在主轴上排列Flex Item的方式， 可接受的值：

- flex-start
- flex-end
- center
- space-between
- space-around
- space-evenly

### align-items

`align-items`属性为 在交叉轴上排列Flex Item的方式， 可接受的值：

- flex-start
- flex-end
- center
- baseline
- stretch

### align-content

`align-content`属性为 在Flex Item为多行时的排列方式， 可接受的值：

- flex-start
- flex-end
- center
- space-between
- space-around
- stretch
- space-evenly

其中`space-evenly`: flex项都沿着主轴均匀分布在指定的对齐容器中。相邻flex项之间的间距，主轴起始位置到第一个flex项的间距,，主轴结束位置到最后一个flex项的间距，都完全一样

## Flex Item 的属性

### align-self

`align-self` 重写在容器上的align-items属性，可接受的值

- auto
- flex-start
- flex-end
- center
- baseline
- stretch

### order

`order` 为Flex Item的排列顺序，默认为0, 越大的值将排在越后面

### flex-grow

`flex-grow` 属性定义了将如何分隔剩余的空间， 默认值为 1, 值不能小于0.

### flex-shrink

`flex-shrink` 属性定义了 当容器空间不足时将如何缩小Flex Item， 默认值为 1, 值不能小于0.

### flex-basis

`flex-basis` 属性定义了 Flex Item的初始大小, 默认为auto


### flex

`flex`属性为 `flex-grow` `flex-shrink`  `flex-basis` 三个的缩写, 如 `flex: 1 1 200px`

**注：当写作`flex: 1;` 这种缩写时，其完整表达式为 `flex: 1 0 0;`**
