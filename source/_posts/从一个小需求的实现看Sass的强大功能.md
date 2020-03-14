---
title: 从一个小需求的实现看Sass的强大功能
date: 2020-03-12 20:08:00
urlname: sass-handbook
categories: ["技术"]
tags: ["Sass", "CSS"]
toc: true
---

最近在深入学习`Sass`的时候，看到了这样一段实现多皮肤的样式代码:

```scss
$themes: (
  default: (
    background-color: #000,
  ),
  light: (
    background-color: #fff,
  ),
);
@mixin themeify {
  @each $theme-name, $theme-map in $themes {
    $theme-map: $theme-map !global;
    body[data-theme='#{$theme-name}'] & {
      @content;
    }
  }
}

@function themed($key) {
  @return map-get($theme-map, $key);
}
.content {
  position: relative;
  @include themeify() {
    background: themed('background-color');
  }
}
```

短短的十几行代码却大有乾坤，涵盖了诸多`Sass`的知识点。这段样式编译成的`css`如下：

```css
.content {
  position: relative; }
  body[data-theme='default'] .content {
    background: #000; }
  body[data-theme='light'] .content {
    background: #fff; }
```

就这样，我们得到了两套不同标签下的背景色样式。那它的实现思路是怎样的？又应用了哪些`Sass`的规则特性？待我一点点拆开细说。

### 变量$

`Sass`中使用`$`符号声明变量方便管理和使用，变量值可以是`数字`、`字符串`、`颜色值`、`布尔值`、`空值(null)`、`数组`(list，以空格或逗号分隔)以及`maps`(相当于对象，用圆括号包裹键值对)。像上文的变量`$themes`就是一个`maps`，定义了多个主题和其对应的样式集合。

变量定义好之后直接在需要的样式中使用:

```scss
$font-color: red;

.artical {
  color: $font-color;
}

// 编译成css
.artical {
  color: red; }
```

### 插值语句#{}

通过 `#{}` 插值语句可以在选择器或属性名中使用变量:

```scss
$name: foo;
$attr: border;
p.#{$name} {
  #{$attr}-color: blue;
}

// 编译成css
p.foo {
  border-color: blue; }
```

#### 作用域

`Sass`变量的作用域只能在当前的层级上有效果，内部定义的变量无法在外部使用:

```scss
$font-color: red;

.artical {
  $font-color: blue;
  color: $font-color;
}

.title {
  color: $font-color;
}

// 编译成css
.artical {
  color: blue; }

.title {
  color: red; }
```

#### 全局关键词!global

`!global`用以将变量声明为全局。

```scss
$font-color: red;

.artical {
  $font-color: blue!global;
  color: $font-color;
}

.title {
  color: $font-color;
}

// 编译成css
.artical {
  color: blue; }

.title {
  color: blue; }
```

### 嵌套规则

编写普通`css`时我们经常会为一些有包含关系的样式重复输入父选择器，`Sass`提供的嵌套规则为我们省去这种烦恼。

```scss
.content{
  background: #fff;
  .box{
    color: #000;
    .title{
      font-size: 24px;
    }
  }
}

// 编译成css
.content {
  background: #fff; }
  .content .box {
    color: #000; }
    .content .box .title {
      font-size: 24px; }
```

`Sass`的编译机制会在遇到嵌套规则块时逐层解套，并将外层的父选择器添加到内层的选择器前，通过空格连接构成后代选择器。

#### 父选择器标识符&

写过`Sass`的人应该知道当我们想给一个元素定义伪类选择器的时候是这样写的:

```scss
.link {
  color: blue;
  &:hover {
    color: red;
  }
}

// 编译成css
.link {
  color: blue; }
  .link:hover {
    color: red; }
```

`&`符号在`Sass中`被称为父选择器标识符。重点来了：

> <font color="#c7254e" size="3">当包含&的嵌套规则块被打开时，它不会像后代选择器那样进行拼接，而是&被父选择器直接替换。如果含有多层嵌套，最外层的父选择器会一层一层向下传递。</font>

因此，当`&`前有其他选择器时，即便是在嵌套内层，编译之后该选择器也不会成为后代选择器:

```scss
.link {
  color: blue;
  .wrap & {
    color: red;
  }
}

// 编译成css
.link {
  color: blue; }
  .wrap .link {
    color: red; }
```

这对于我们想要声明一些特殊情况下的样式时非常有帮助，也是文章开头实现多皮肤的关键。

### 函数

`Sass`定义了非常丰富的函数供不同需求使用，比如针对`maps`变量可以通过`map-get($map, $key) `方法获取属性值:

```scss
$font-weights: ("regular": 400, "medium": 500, "bold": 700);

map-get($font-weights, "medium"); // 500
```

- `map-has-key($map, $key)`检查是否含有某个属性
- `map-keys($map)`获取属性名集合
- `map-merge($map1, $map2)`合并两个`maps`
- `map-values($map)`获取属性值...等等

更多函数方法[看这里](https://sass.bootcss.com/documentation/modules)。

#### @function

`Sass`还支持自定义函数，可传入参数:

```scss
$grid-width: 40px;

@function grid-width($n) {
  @return $n * $grid-width;
}

#sidebar { width: grid-width(5); }

// 编译成css
#sidebar {
  width: 200px; }
```

### @mixin和@include

`@mixin`（混合样式）用于定义一段可以重复使用的样式，并且可以接受参数引入变量，参数支持定义默认值。使用时通过`@include`引入：

```scss
@mixin border($color, $width: 1px, $style: solid) {
  border: $width $style $color;
}

.artical {
  @include border(#ccc);
}

// 编译成css
.artical {
  border: 1px solid #ccc; }

// 还可以使用关键词参数，打乱参数顺序使用
.artical {
  @include border($width: 2px,$color:#ccc);
}
// 编译后
.artical {
  border: 2px solid #ccc; }
```

不仅仅是传参，在引用混合样式的时候，可以先将一段代码导入到混合指令中，然后再输出混合样式，额外导入的部分将出现在 `@content` 标志的地方：

```scss
@mixin border($color, $width: 1px, $style: solid) {
  border: $width $style $color;
  @content;
}

.artical {
  @include border(#ccc) {
    .title {
      color: #000;
    }
  }
}

// 编译后
.artical {
  border: 1px solid #ccc; }
  .artical .title {
    color: #000; }
```

### @each遍历

`@each` 指令的格式是 `$var in <list>`, `$var` 可以是任何变量名，比如 `$length` 或者 `$name`，而`<list>`是一连串的值，也就是值列表。`$var`的数量可以是一个或者多个，取决于`list`中每一组值的数量。`@each`对于遍历`maps`变量同样适用，`$var`对应键名和键值。

且看搬运自官网的例子：

```scss
// 列表
@each $animal in puma, sea-slug, egret, salamander {
  .#{$animal}-icon {
    background-image: url('/images/#{$animal}.png');
  }
}
// 编译成css
.puma-icon {
  background-image: url("/images/puma.png"); }

.sea-slug-icon {
  background-image: url("/images/sea-slug.png"); }

.egret-icon {
  background-image: url("/images/egret.png"); }

.salamander-icon {
  background-image: url("/images/salamander.png"); }

// 二维列表
@each $animal, $color, $cursor in (puma, black, default),
                                  (sea-slug, blue, pointer),
                                  (egret, white, move) {
  .#{$animal}-icon {
    background-image: url('/images/#{$animal}.png');
    border: 2px solid $color;
    cursor: $cursor;
  }
}

// 编译成css
.puma-icon {
  background-image: url("/images/puma.png");
  border: 2px solid black;
  cursor: default; }

.sea-slug-icon {
  background-image: url("/images/sea-slug.png");
  border: 2px solid blue;
  cursor: pointer; }

.egret-icon {
  background-image: url("/images/egret.png");
  border: 2px solid white;
  cursor: move; }

// maps
@each $header, $size in (h1: 2em, h2: 1.5em, h3: 1.2em) {
  #{$header} {
    font-size: $size;
  }
}
// 编译成css
h1 {
  font-size: 2em; }

h2 {
  font-size: 1.5em; }

h3 {
  font-size: 1.2em; }
```

### 拨云见日

介绍完了这么多的知识点，开头的需求是如何实现的就很明了了。

首先定义几组主题和对应的样式：

```scss
$themes: (
  default: (
    background-color: #000,
  ),
  light: (
    background-color: #fff,
  ),
);
```

在混入样式中遍历主题，并将各组主题样式声明为全局变量，以便导入混入样式时可通过自定义函数获取属性值。

```scss
@mixin themeify {
  @each $theme-name, $theme-map in $themes {
    $theme-map: $theme-map !global;
    body[data-theme='#{$theme-name}'] & {
      @content;
    }
  }
}

@function themed($key) {
  @return map-get($theme-map, $key);
}
.content {
  position: relative;
  @include themeify() {
    background: themed('background-color');
  }
}
```

有了这些基础再通过`插值语句#{}`和父选择器标识符`&`生成`body`标签不同的主题属性选择器样式就最终实现了多皮肤的需求，给强大的`Sass`竖大拇指！

