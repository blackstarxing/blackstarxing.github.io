---
title: 探访JS作用域
date: 2018-10-18 21:38:50
urlname: javascript-scope
categories: ["技术"]
tags: ["JS"]
toc: true
---

### 引子

文章开始，先用一道我曾经栽过跟头的面试题来引出今天的主题：

``` javascript
var a = 1
function scopeA() {
    var a = 2
    scopeB()
}
function scopeB() {
    console.log(a)
}
scopeA()
```

想想看这段代码的输出结果是1还是2？为什么？

### 什么是作用域？

几乎所有编程语言最基本的功能之一，就是能够储存变量当中的值并且能在之后对这个值进行访问或修改。而作用域就是一套规则，用于确定在何处以及如何查找变量。

### ES5作用域

ES5中，作用域分为全局作用域以及函数作用域。函数作用域是指在函数内声明的所有变量在函数体内始终是可见的，在函数外部无法访问，由此变量也就分成了全局变量和局部变量。

``` javascript
var a = 1
function foo(){ 
    var b = 2 
    console.log(a) // 1
    console.log(b) // 2
} 
foo();
console.log(a) // 1
console.log(b) // Uncaught ReferenceError: b is not defined
```

上面的代码，a就是全局变量，可以被任意访问，而b是只属于函数foo的内部变量，所以在外部访问会报错。

而由于JavaScript没有块级作用域，因此在`for`、`if`等代码块中声明的变量也属于全局变量。

``` javascript
for(var i = 0;i < 10;i++){
    
}
console.log(i) // 10

if(true){
    var a = 1
}
console.log(a) // 1
```

### 作用域链

当一个块或函数嵌套在另一个块或函数中时，就发生了作用域的嵌套。当在当前作用域中无法找到某个变量时，引擎就会在外层嵌套的作用域中继续查找，直到找到该变量，或抵达最外层的作用域（也就是全局作用域）为止。作用域链是函数被创建的作用域中对象的集合，它保证了对执行环境中有权访问的所有变量和函数的有序访问。

回到文章开头提到的那段代码，其运行结果是`1`。

虽然函数`scopeB`是在`scopeA`中调用的，但是函数运行在它们被定义的作用域里，而不是它们被执行的作用域里，因此当`scopeB`内部找不到变量a时，再向外查找就是查询全局作用域，此时变量a的值就是1。

### 变量提升

来看一段代码：

``` javascript
function foo(){
    console.log(a) // undefined
    var a = 1
}
foo()
```

为什么a的打印结果是`undefined`？在JavaScript中，函数及变量的声明都将被提升到函数的最顶部。因此上面的代码等同于：

```javascript
function foo(){
    var a
    console.log(a) // undefined
    a = 1
}
foo()
```

a打印时只声明未赋值，因此结果是`undefined`。

### 函数声明和函数表达式

定义函数的方式有两种，一种是函数声明，另一种是函数表达式。函数声明会将函数提升到最前面，成为全局函数，并且一定要声明指定函数名，而函数表达式可以不用声明函数名用作匿名函数，但是要在赋值后才可以调用；函数表达式后面可以直接跟`()`调用，函数声明不可以。

```javascript
// 函数声明
foo() // 1
function foo(){
    console.log(1)
}
// 函数表达式
bar() // Uncaught TypeError: bar is not a function
var bar = function(){
    console.log(2)
}
```

### 立即执行函数（IIFE）

虽然函数作用域可以将内部变量隐藏起来，但是由于必须声明一个具名函数，这个变量名会污染外部作用域，而且必须显式的通过方法名调用才可以执行。

幸好我们只需在函数外面加一对圆括号就可以解决这些问题。

``` javascript
(function foo(){
    var a = 3
    console.log(a) // 3
})()
```

加上圆括号就把函数声明变成了函数表达式，后面再接一对圆括号表示立即执行这个函数。这种方式定义的函数就是立即执行函数。

区分函数声明和表达式最简单的方法是看`function`关键字出现在声明中的位置。如果`function`是声明中的第一个词，那么就是一个函数声明，否则就是一个函数表达式。

`(function foo(){ .. }) `作为函数表达式意味着foo只能在..所代表的位置中被访问，外部作用域则不行。foo变量名被隐藏在自身中意味着不会非必要地污染外部作用域。其实立即执行函数模仿了块级作用域的效果。

### 块级作用域

虽然函数作用域可以隐藏内部变量，但函数并不是唯一的作用域单元，变量的声明应该距离使用的地方越近越好，有些变量我们只希望在部分代码块`{..}`中才可以使用，而声明为全局变量也许会带来一些不必要的后果。好在ES6为我们带来了`let`和`const`关键字，这让我们可以创建块级作用域。

``` javascript
{
    console.log(a) // Uncaught ReferenceError: a is not defined
    let a = 1
    console.log(a) // 1
}
console.log(a) // Uncaught ReferenceError: a is not defined
```

这样，通过`let`声明的变量就只在所在代码块内有效，而不会泄漏到全局，也不会提升到块的顶部。