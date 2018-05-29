---
layout: post
title: "scss 复习"
date: 2018-05-21 00:00:00
categories: css
comments: true
---

# Comment

```scss
  // 这个备注不会
  // 在编译后的css文件中
  // 出现

  /* 这个备注会被编译到目标css文件 */
```

# Import

```scss
  // 包括 _buttons.sass buttons.sass _buttons.scss button.scss
  @import 'buttons';
```

# Nesting

```scss
  .content {
    border: 1px solid #ccc; padding: 20px;
    h2 {
      font-size: 3em;
      margin: 20px 0;
    }
    p {
      font-size: 1.5em;
      margin: 15px 0;
    }
  }
```

### Nesting Properties

```scss
  // scss
  .btn {
    text: {
        decoration: underline;
        transform: lowercase;
      }
    }
  // 编译后css
  .btn {
    text-decoration: underline;
    text-transform: lowercase;
  }
```

### Parent Selector

```scss
  a {
    &:hover {
      color: #777;
    }
    &:active {
      color: #888;
    }
  }

  .sidebar {
    font-size: 14px;
    & .link {
      color: #fff;
    }
    // .sidebar .link
  }

  .sidebar {
    float: left;
    width: 100px;
    h2 {
      color: #000;
      .user & {
        color: #ddd;
        // .user .sidebar h2
      }
    }
  }
// 嵌套层次需要适当，不应超过3-4级嵌套
```

# Variable

```scss
  $base-color: #ddd;

  .sidebar {
    border: $base-color 1px solid;
    p {
      color: $base-color
    }
  }
```

### 默认 !default option in Variable

```scss
  $content: "My Content";
  $content: "Will not overwrite" !default

  h1:before {
    content: $content;
  }

  // app.scss
  $rounded: 5px;
  @import "buttons";

  // _buttons.scss
  $rounded: 3px !default;
  .btn-a {
    border-radius: $rounded;
    color: #777;
  }
  .btn-b {
    border-radius: $rounded;
    color: #222;
  }
```

### Variable Scope
- Variables set inside a declaration (within { }) aren't usable outside that block
- Setting new values to variables set outside a declaration changes future instances

```scss
  $color-base: #777777;
  .sidebar {
    $color-base: #222222;
    background: $color-base; // #222222
  }
  p {
    color: $color-base; // #222222
  }
```

- 使用#{$variable}

  ```scss
    $side: top;
    sup {
      position: relative;
      #{$side}: -0.5em;
    }
    .callout-#{$side} {
      background: #777;
    }
  ```

# Mixin

```scss
  // mixin 需要在 include之前定义
  @mixin button {
    border: 1px solid #ddd;
    font-size: 1em;
    text-transform: uppercase;
  }

  .btn-a {
    @include button;
    background-color: #eee;
  }
  .btn-b {
    @include button;
    background-color: #bbb;
  }
```

### Mixin with arguments

```scss
  @mixin box-sizing($x: border-box) {
    -webkit-box-sizing: $x;
    -moz-box-sizing: $x;
    box-sizing: $x;
  }

  .box1 {
    @include boxsizing; // border-box;
  }
  .box2 {
    @include boxsizing(border-box); // border-box
  }
  .box3 {
    @include boxsizing(content-box) // content-box
  }


  @mixin button($radius, $color: #000) {
    border-radius: $radius;
    color: $color;
  }

  .btn-a {
    @include button(4px);
  }


  @mixin transition($val...) {
    -webkit-transition: $val;
    -moz-transition: $val;
    transition: $val;
  }
  .btn-a {
    @include transition(color 0.3s ease-in, background 0.5s ease-out);
  }



  @mixin button($radius, $color) {
    border-radius: $radius;
    color: $color;
  }

  $properties: 4px, #000;

  .btn-a {
    @include button($properties...); // border-radius: 4px; color: #000;
  }


  @mixin highlight($color, $side) {
    border-#{$side}-color: $color;
  }

  .btn-a {
    @include highlight(#ff0, right); // border-right-color: #ff0;
  }

```

# Extend

``` scss
  .btn-a {
    background: #777;
    border: 1px solid #ccc;
    font-size: 1em;
    text-transform: uppercase;
  }

  .btn-b {
    @extend .btn-a;
    background: #ff0;
  }

  .sidebar .btn-a {
    text-transform: lowercase;
  }

  // =>
  .btn-a,
  .btn-b {
    background: #777;
    border: 1px solid #ccc;
    font-size: 1em;
    text-transform: uppercase;
  }
  .btn-b {
    background: #ff0;
  }

  .sidebar .btn-a,
  .sidebar .btn-b {
    // btn-b is also scoped here.
    text-transform: lowercase;
  }

```

### placeholder selectors

```scss
  %btn {
    background: #777;
    border: 1px solid #ccc;
    font-size: 1em;
    text-transform: uppercase;
  }
  .btn-a {
    @extend %btn;
  }
  .btn-b {
    @extend %btn;
    background: #ff0;
  }
  .sidebar .btn-a {
    text-transform: lowercase;
  }

  // =>

  .btn-a,
  .btn-b {
    background: #777;
    border: 1px solid #ccc;
    font-size: 1em;
    text-transform: uppercase;
  }
  .btn-b {
    background: #ff0;
  }
  .sidebar .btn-a {
    text-transform: lowercase;
  }
```

# Function

```scss
  @function fluidize($target, $context) {
    @return ($target / $context) * 100%; }
  }
  .sidebar {
    width: fluidize(350px, 1000px);
  }

  // =>
  .sidebar {
    width: 35%;
  }
```

# If

```scss

$theme: pink;
header {
  @if $theme == dark {
    background: #000;
  } @else if $theme == pink {
    background: pink;
  } @else {
    background: #fff;
  }
}
// =>
header {
  background: pink;
}

```

# Each

``` scss
$authors: nick aimee dan drew;

@each $author in $authors {
  .author-#{$author} {
    background: url(author-#{$author}.jpg);
  }
}

// =>
.author-nick {
  background: url(author-nick.jpg);
}
.author-aimee {
  background: url(author-aimee.jpg);
}
.author-dan {
  background: url(author-dan.jpg);
}
.author-drew {
  background: url(author-drew.jpg);
}

```

```scss
// for
.item {
  position: absolute;
  right: 0;
  @for $i from 1 through 3 {
    &.item-#{$i} {
      top: $i * 30px;
    }
  }
}

// =>
.item {
  position: absolute;
  right: 0;
}
.item.item-1 {
  top: 30px;
}
.item.item-2 {
  top: 60px;
}
.item.item-3 {
  top: 90px;
}

```

```scss
// while

$i: 2;
.item {
  position: absolute;
  right: 0;
  @while $i <= 6 {
    }
  &.item-#{$i} {
    top: $i * 30px;
  }
  $i: $i + 2;
}

// =>
.item {
  position: absolute;
  right: 0;
}
.item.item-2 {
  top: 60px;
}
.item.item-4 {
  top: 120px;
}
.item.item-6 {
  top: 180px;
}
```

```scss
// Mixin In
@mixin button($color, $rounded: true) {
  color: $color;
  @if $rounded == true {
    border-radius: 4px;
  }
}
.btn-a {
  @include button(#000, false);
}
.btn-b {
  @include button(#333);
}

// =>
.btn-a {
  color: black;
}
.btn-b {
  color: #333333;
  border-radius: 4px;
}

// if $rounded => $rounded不是false或者null时都为真
```

# Math

- sass 返回数字默认精度为小数点后5位

```scss
  font: normal 2em/1.5 Helvetica, sans-serif;

  $family: "Helvetica " + "Neue";  // "Helvetica Neue"

  $family: 'sans-' + serif  // 'sans-serif'
  $family: sans- + 'serif'  // sans-serif

  h2 {
    font-size: 10px + 4pt;  // font-size: 15.33333px;
  }

  h2 {
    font-size: 10px + 4em;  // error 计算单位不适用
  }
```

- pre defined math utilities
  - `round($number)` - round to closest whole number ceil($number) - round up
  - `floor($number)` - round down
  - `abs($number)` - absolute value
  - `min($list)` - minimum list value
  - `max($list)` - maximum list value percentage($number) - convert to percentage

```scss
h2 {
  line-height: ceil(1.2); // line-height: 2;
}

.sidebar {
  width: percentage(350px/1000px);  // width: 35%;
}

$context: 1000px;
.sidebar {
  width: percentage(450px/$context);  // width: 45%;
}
```


# color with math

```scss
$color-base: #333333;
.addition {
  background: $color-base + #112233;  // background: #445566;
}
.subtraction {
  background: $color-base - #112233;  // background: #221100;
 }
.multiplication {
  background: $color-base * 2;  // background: #666666;
}
.division {
  background: $color-base / 2;  // background: #191919;
}


$color: #333333;
.alpha {
  background: rgba($color,0.8);  // background: rgba(51,51,51,0.8);
}
.beta {
  background: rgba(#000,0.8);  // background: rgba(0,0,0,0.8);
}


$color: #333;
.lighten {
  color: lighten($color, 20%);  // background: #666666;
}
.darken {
  color: darken($color, 20%);  // background: black;
}

$color: #87bf64;
.saturate {
  color: saturate($color, 20%);  // background: #82d54e;
}
.desaturate {
  color: desaturate($color, 20%);  // background: #323130;
}



.mix-a {
  color: mix(#ffff00, #107fc9);  // background: #87bf64;
}
.mix-b {
  color: mix(#ffff00, #107fc9, 30%);  // background: #57a58c;
}




$color: #87bf64;
.grayscale {
  color: grayscale($color);  // color: #929292;
}
.invert {
  color: invert($color);  // color: #78409b;
}
.complement {
  color: complement($color);  // color: #9c64bf;
}
```

- More Functions: http://sass-lang.com/documentation/Sass/Script/Functions.html


# Responsive

```scss
.sidebar {
  border: 1px solid #ccc;
  @media (min-width: 700px) {
    float: right;
    width: 30%;
  }
}

// =>
.sidebar {
  border: 1px solid #ccc;
}
@media (min-width: 700px) {
  .sidebar {
    float: right;
    width: 30%;
  }
}



@mixin respond-to($media) {
  @if $media == tablet {
    @media (min-width: 700px) {
      @content
    }
  }
}
.sidebar {
  border: 1px solid #ccc;
  @include respond-to(tablet) {
    float: right;
    width: 30%;
  }
}


@mixin respond-to($query) {
  @media (min-width: $query) {
    @content
  }
}
.sidebar {
  border: 1px solid #ccc;
  @include respond-to(900px) {
    float: right;
    width: 30%;
  }
}



@mixin respond-to($val, $query) {
  @media ($val: $query) {
    @content
  }
}
.sidebar {
  border: 1px solid #ccc;
  @include respond-to(max-width, 600px) {
    float: right;
    width: 30%;
  }
}

```
