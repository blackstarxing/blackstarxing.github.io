---
title: 解惑JavaScript中的this指向问题
subtitle: this的绑定取决于函数的调用方式
date: 2018-11-01 21:10:25
urlname: javascript-this-disabuse
categories: ["技术"]
tags: ["JS"]
toc: true
---

### 消除错误认知

首先，我们必须要消除的两个关于this的误解：

> * 指向函数自身
> * 指向函数作用域

需要牢记的是，this在任何情况下都不指向函数的词法作用域，`this的绑定取决于函数的调用方式`。

### 绑定规则

既然明确了this的绑定和调用方式有关，我们就可以根据不同的情况来判断指向结果，我们可以认为这是一组绑定规则。

#### 1、默认绑定

这适用于最最普通的函数调用，没有其余规则掺杂，仅仅针对于独立函数。

``` javascript
function foo(){
    console.log(this) //window
}
foo()
```

可以看到，此时函数foo中的this指向的是window。这是因为默认绑定规则下，this指向全局。但这只是在非严格模式下，如果是在严格模式下，this会绑定到undefined。

#### 2、隐式绑定

当函数的调用位置有上下文对象（也就是通过对象方法调用）时，this指向这个上下文对象。

``` javascript
function foo() {
    console.log( this.a )
}
var obj = {
    a: 2,
    foo: foo
}
obj.foo() // 2
```

而当出现多级调用时，this绑定的是离它最近的一个上下文对象。

``` javascript
function foo() {
    console.log( this.a )
}
var obj1 = {
    a: 1,
    foo: foo
}
var obj2 = {
    a: 2,
    obj1: obj1
}
obj2.obj1.foo() // 1
```

注意该规则针对于对象方法调用，像下面这样实际还是默认绑定：

``` javascript
var a = 1
var obj = {
    a:2,
    foo:function(){
        var fn = function(){
            console.log(this.a) // 1
        }
        fn() // fn没有对象调用，this绑定全局
    }
}
obj.foo()
```

可是这种隐式绑定this会丢失绑定的对象，常见的两种情况是引用赋值和回调函数：

（1）引用赋值

``` javascript
function foo() {
    console.log( this.a )
}
var obj = {
    a: 2,
    foo: foo
}
var bar = obj.foo // 函数别名！
var a = 1
obj.foo() // 2
bar() // 1
```

这里看起来bar()和obj.foo()是一样的，但其实bar引用的是foo函数本身，因此此时的bar()其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

（2）回调函数

``` javascript
function foo() {
    console.log( this.a )
}
function doFoo(fn) {
    // fn 其实引用的是 foo
    fn() // <-- 调用位置！
}
var obj = {
    a: 2,
    foo: foo
}
var a = 1 // a 是全局对象的属性
doFoo( obj.foo ) // 1
```

看到了吧，传入回调函数的方法绑定的this也应用了默认绑定规则，这也解释了为什么`setTimeout`方法中回调函数的this会指向window，因为其运行在与所在函数完全分离的执行环境上。

#### 3、显式绑定

若要强制的调用不属于对象内部的方法，将this绑定到对象上面可以使用`call()`和`apply()`方法，这两个方法都可以实现改变this指向，它们接受的第一个参数都是要绑定this的对象。区别在于：`call()`从第二个参数开始接受传递给方法的参数，可以有多个，而`apply()`最多接受两个参数，第二个参数为一个数组，用来给方法传参。

``` javascript
function foo() {
    console.log( this.a )
}
var obj = {
    a:2
}
foo.call( obj ) // 2
```

函数foo中的this被强制的绑定到了obj这个对象上。

还可以使用`bind()`方法创建一个新的函数，`bind()`方法中指定一个参数作为this上下文。

#### 4、new绑定

最后一种绑定规则，如果在一个函数前面带上new来调用，那么将会创建一个新对象，同时函数中的this会绑定到这个新对象上。在 JavaScript 中，把这些使用new操作符时被调用的函数称为构造函数。

``` javascript
function foo(a) {
    this.a = a
    console.log(this)
}
var bar = new foo(2) // foo {a: 2}
```

如此，foo中的this就被绑定到了bar上。

### 优先级

介绍了各种this绑定规则，那么当不同的调用规则混合使用时，我们就需要判断优先级，而new绑定高于显式绑定高于隐式绑定高于默认绑定。

因此我们可以得出以下的判断方法：

>1、由 new 调用？绑定到新创建的对象。
2、由 call 或者 apply（或者 bind）调用？绑定到指定的对象。
3、由上下文对象调用？绑定到那个上下文对象。
4、默认：在严格模式下绑定到 undefined，否则绑定到全局对象。

### 箭头函数中的this

不同于前面介绍的四种绑定规则，箭头函数内部的this是词法作用域，由上下文确定，因此箭头函数中的this指向的是定义时的this，而不是执行时的this，并且箭头函数的绑定无法修改。

``` javascript
var obj = {
    a:1,
    foo:function(){
        setTimeout(()=>{
            // this指向obj
            console.log(this.a)
        },1000)
    }
}
var a = 2
obj.foo() // 1而不是2
```

``` javascript
var obj = {
    a:1,
    foo:()=>{
        // this指向window
        console.log(this.a)
    }
}
var a = 2
obj.foo() // 2而不是1
```

上述两段代码都是使用了箭头函数，可为什么结果不尽相同？

要记住，箭头函数中的this是根据定义时的上下文决定的，第一段代码中setTimeout中的箭头函数定义在foo中，而foo的上一级是obj，所以this指向obj。而第二段代码的箭头函数是定义在obj中，obj的上一级是全局，因此this指向全局。

***

到此我们就已经介绍了this的全部绑定情况，虽然例子举得简单，但是明确了判断规则后，相信你再不会被小小的this难住。

*&#42;本文根据**《你不知道的JavaScript（上）》**第二部分第二章总结整理*

