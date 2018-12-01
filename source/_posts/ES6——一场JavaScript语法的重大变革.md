---
title: ES6——一场JavaScript语法的重大变革
subtitle: less is more
date: 2018-11-28 23:21:08
urlname: es6-change
cover: "https://s1.ax1x.com/2018/11/28/FZneED.jpg"
categories: ["技术"]
tags: ["JS","ES6"]
toc: true
---

### 前言

ES5标准发布后，JavaScript的发展步伐越来越快，新的特性和语法形式亟待标准化。而在经历了各种波折后，千呼万唤的ECMAScript 6（简称ES6）作为JavaScript语言的下一代标准终于在2015年发布。

对于JavaScript这门语言来说，ES6 是一次激进的飞跃。简洁高效的语法解放了开发者，同时也弥补了一些最初的设计缺陷。可以说这是一场JavaScript语法的重大变革。

那么ES6究竟为我们带来了哪些新的特性？它又颠覆哪些我们之前固有的语法习惯？

下面这张思维导图概括了ES6的一些核心特性，虽不完全但足够日常使用。

![es6_mind](https://s1.ax1x.com/2018/11/29/FZsRBV.png)

*<font color="#bbb" size="3">思维导图来自@[浪里行舟](https://github.com/ljianshu/Blog)，已获得使用许可</font>*

### 箭头函数

箭头函数是ES6使用率最高的特性之一。从字面上看，箭头函数简化了定义函数的写法，使用`=>`替代`function`。

```javascript
// ES5
function foo(x,y) { 
    return x + y
} 
// ES6
var foo = (x,y) => x + y
```

箭头函数定义包括一个参数列表（零个或多个参数，如果参数个数不是一个的话要用`( .. )`包围起来），然后是标识` =>`，函数体放在最后。 

所以，在前面的代码中，箭头函数就是`(x,y) => x + y`这一部分，然后这个函数引用被赋给变量 foo。 

只有在函数体的表达式个数多于1个，或者函数体包含非表达式语句的时候才需要用`{ .. } `包围。如果只有一个表达式，并且省略了包围的`{ .. }`的话，则意味着表达式前面有一个隐含的`return`，就像前面代码中展示的那样。

由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错。 这里列出了几种不同形式的箭头函数： 

``` javascript
var f1 = () => 12
var f2 = x => x * 2
var f3 = (x,y) => { 
    var z = x * 2 + y
    y++
    x *= 3
    return (x + y + z) / 2
}
var f4 = (x,y) => ({ x:x, y:y })
```

箭头函数**总是**函数表达式；并不存在箭头函数声明。

除了简化写法，箭头函数的主要设计目的是以特定的方式改变`this`的行为特性。在前面的[解惑JavaScript中的this指向问题](/2018-11-01-javascript-this-disabuse.html)中我们有介绍，ES5中`this`指向取决于函数调用方式，而箭头函数中的`this`不一样，它指向定义时的对象，这也正是多数情况下我们所希望的。

``` javascript
// ES5中使用var self = this这一hack方法绑定this
var obj = {
    a:1,
    foo:function(){
        var self = this
        setTimeout(function(){
            // 希望获取obj中的a
            console.log(self.a)
        },1000)
    }
}
// 箭头函数
var obj = {
    a:1,
    foo:function(){
        setTimeout(()=>{
            // 没问题this指向obj
            console.log(this.a)
        },1000)
    }
}
```

### let和const

我们知道ES5中只有全局作用域和函数作用域，并没有块级作用域的概念（有关作用域看[这里](/2018-10-18-javascript-scope.html)），这会导致几个问题：

1、变量可能会发生冲突并覆盖变量值

2、用来计数的循环变量泄露为全局变量，比如在for循环中。

如果在ES5中需要创建一个块级作用域可以使用立即调用函数表达式（IIFE），IIFE中的变量和函数会在在执行后销毁，防止污染到外部作用域。

ES6为我们带来了`let`和`const`两个命令，我们可以利用它们创建绑定到任意块的声明，我们只需要一对`{ .. }`就可以创建一个作用域。不像使用var那样声明的变量总是归属于包含函数（即全局，如果在最顶层的话）作用域，`let`声明的变量只在其所在的代码块内有效，且不存在变量提升。

``` javascript
var a = 2
{
    console.log( a ) // Uncaught ReferenceError: a is not defined
    let a = 3
    console.log( a ) // 3
}
console.log( a ) // 2
```

和`let`差不多，只不过`const`声明一个只读的常量。一旦声明，常量的值就不能改变。实际应用中，使用`const`定义变量也能够明白地告诉后来的开发者该变量不可修改赋值。

### 模板字符串

ES6以前，我们定义字符串模板时，需要使用`+`拼接字符串、变量以及进行换行：

``` javascript
var name = 'blackstar'
var str = 'hello ' + name + ','
	'nice to meet you!'
```

在ES6中我们使用反引号** `** 作为标识，它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量甚至是函数调用。

``` javascript
function upper(s) {
    return s.toUpperCase()
}
var name = 'blackstar'
// 模板字符串会保留所有的空格和缩进
var str = `hello ${upper(name)},
	nice to meet you!`
```

### 解构赋值

ES6允许按照一定模式，从数组和对象中提取值对变量进行赋值，这被称为解构。

#### 数组解构赋值

``` javascript
var a = 1
var b = 2
var c = 3
// 无须再像上面那样一个一个赋值
var [a,b,c] = [1,2,3]
```

解构赋值支持嵌套，数组的解构赋值按照位置对应取值：

``` javascript
let [foo, [[bar], baz]] = [1, [[2], 3]]
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"]
third // "baz"

let [x, , y] = [1, 2, 3]
x // 1
y // 3

let [x, y] = ['a']
x // "a"
y // undefined
```

解构赋值允许指定默认值。

``` javascript
let [x, y = 'b'] = ['a'] // x='a', y='b'
```

#### 对象解构赋值

不同于数组按照位置解构赋值，对象的解构赋值是按照属性名来的。变量名须与属性名相同才可以取到值。

``` javascript
let { foo, baz } = { foo: "aaa", bar: "bbb" }
foo // "aaa"
baz // undefined
```

如果变量名与属性不一致，需要使用如下写法：

``` javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' }
baz // "aaa"
```

对象解构赋值同样支持嵌套和指定默认值。

有一点很重要需要牢记，如果是给已经声明的变量赋值，需要使用`()`包裹，不然`{}`会被当做代码块。

``` javascript
let x
({x} = {x: 1})
```

#### 字符串解构赋值

``` javascript
const [a, b, c, d, e] = 'hello'
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

#### 函数参数解构赋值

``` javascript
function add([x, y]){
    return x + y
}

add([1, 2]) // 3
```

### 默认参数值

JavaScript最常见的一个技巧就是关于设定函数参数默认值的。多年以来我们实现这一点的方式是这样的：

``` javascript
function foo(x,y) {
    x = x || 11
    y = y || 31
    console.log( x + y )
}
foo() // 42
foo( 5, 6 ) // 11
foo( 5 ) // 36
foo( null, 6 ) // 17
```

当然，如果你之前使用过这个模式，就会知道它很有用，但同时又有点危险，比如，如果对于一个参数你需要能够传入被认为是false（假）的值。考虑：

``` javascript
foo( 0, 42 ) // 53 <--哎呀，并非42
```

为什么？因为这里0为假，所以`x || 11`结果为11，而不是直接传入的0。要修正这个问题，有些人会选择增加更多的检查，就像下面这样：

``` javascript
function foo(x,y) {
    x = (x !== undefined) ? x : 11
    y = (y !== undefined) ? y : 31
    console.log( x + y )
}
foo( 0, 42 ) // 42
foo( undefined, 6 ) // 17
```

那么ES6新增的一个有用的语法可以改进为缺失参数赋默认值的流程。

``` javascript
function foo(x = 11, y = 31) {
    console.log( x + y )
}
foo() // 42
foo( 5, 6 ) // 11
foo( 0, 42 ) // 42
foo( 5 ) // 36
foo( 5, undefined ) // 36 <-- 丢了undefined
foo( 5, null ) // 5 <-- null被强制转换为0
foo( undefined, 6 ) // 17 <-- 丢了undefined
foo( null, 6 ) // 6 <-- null被强制转换为0
```

函数默认值可以不只是简单值，它们可以是任意合法表达式，甚至是函数调用。但是需要注意由于参数变量是默认声明的，所以不能用`let`或`const`再次声明。

``` javascript
function foo(x = 5) {
    let x = 1 // error
    const x = 2 // error
}
```

参数默认值也可以与解构赋值的默认值结合起来使用。

``` javascript
function foo({x, y = 5}) {
    console.log(x, y)
}
foo({}) // undefined 5
foo({x: 1}) // 1 5
foo({x: 1, y: 2}) // 1 2
foo() // TypeError: Cannot read property 'x' of undefined
```

### `...`运算符

ES6 引入了一个新的运算符`...`，通常称为spread或rest（展开或收集）运算符，取决于它在哪 / 如何使用。

#### 扩展运算符

当`...`用在数组或字符串之前时，它会把这个变量“展开”为各个独立的值。

``` javascript
console.log(...[1, 2, 3])
// 1 2 3
```

这一特性也方便我们替代`concat()`方法来合并数组.

``` javascript
var a = [2,3,4]
var b = [ 1, ...a, 5 ]
console.log( b ) // [1,2,3,4,5]
```

### rest参数

作为扩展运算符的反向行为，`...`可以用于获取函数的多余参数并且收集到一起成为一个数组，这样就不需要使用arguments对象了。

``` javascript
function foo(x, y, ...z) {
    console.log( x, y, z )
}
foo( 1, 2, 3, 4, 5 ) // 1 2 [3,4,5]
```

对于上面的参数`z`，我们称其为rest参数。`注意：rest参数只能作为最后一个参数`。

### 数组扩展

#### find()和findIndex()

`find()`方法用于找出第一个符合条件的数组成员。它的参数是一个回调函数，回调函数可接受三个参数，依次为当前的值、当前的位置和原数组。所有数组成员依次执行该回调函数，直到找出第一个返回值为`true`的成员，然后返回该成员。如果没有符合条件的成员，则返回`undefined`。

``` javascript
[1, 4, -5, 10].find((n) => n < 0)
// -5
```

`findIndex()`则是返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回`-1`。

#### entries()，keys()和values()

ES6 提供三个新的方法`entries()`，`keys()`和`values()`用于遍历数组。`keys()`是对键名的遍历、`values()`是对键值的遍历，`entries()`是对键值对的遍历。

### Set

ES6提供了新的数据结构Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。

``` javascript
const set = new Set([1, 2, 3, 4, 4])
console.log(set) // Set(4) {1, 2, 3, 4}
```

### Object.assign()

`Object.assign()`方法用于对象的合并，将源对象的所有可枚举属性，复制到目标对象上。但如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。

``` javascript
const target = { a: 1, b: 1 }
const source1 = { b: 2, c: 2 }
const source2 = { c: 3 }
Object.assign(target, source1, source2)
console.log(target) // {a:1, b:2, c:3}
```

### for..of 循环

ES6在把JavaScript中我们熟悉的`for`和`for..in`循环组合起来的基础上，又新增了一个`for..of`循环，`for...of`循环可以使用的范围包括数组、Set和Map结构、某些类似数组的对象（比如arguments对象、DOM NodeList 对象）、Generator对象，以及字符串。

`for..in`在数组的键/索引上循环，而`for..of`在值上循环。

``` javascript
var a = ["a","b","c","d","e"];
for (var idx in a) {
    console.log( idx )
}
// 0 1 2 3 4
for (var val of a) {
    console.log( val )
}
// "a" "b" "c" "d" "e"
```

### Promise

我们在处理异步的时候通常会使用回调来处理流程，可这样会比较混乱，也不利于代码的阅读和维护(可能形成回调地狱)，而ES6为我们提供了一个名为`Promise`的范式，可以不把回调事先传给异步任务，而是给我们提供了解其任务何时结束的能力，然后由我们自己的代码来决定下一步做什么。

```javascript
let promise = new Promise(function(resolve, reject) {
    // ... some code
    if(/* 异步操作成功 */){
        resolve(value)
    }else{
        reject(error)
    }
})
promise.then(function(value) {
    // success
}, function(error) {
    // failure
})
```

有关`Promise`更详细的内容前面的文章[JS单线程与异步](/2018-09-05-javascript-singlethread-asynchronous.html)中有介绍。

### 模块化

在ES6中我们可以通过`import`和`export`命令来帮助我们实现模块化。`export`用来导出变量或函数，`import`则是加载。

``` javascript
// 几种导出方式
export function foo() {
 // ..
} 

function foo() {
 // ..
}
var awesome = 42
var bar = [1,2,3]
export { foo, awesome, bar }

function foo() { .. }
export { foo as bar } // 导出时重命名

// 加载方式
import foo from "foo"
// 或者：
import { foo as bar } from "foo"
```

好了，有关ES6的总结就先写到这里，其实它还有很多特性值得我们去发掘使用，但是把最常使用的用精用熟才是硬道理。

*<font color="#bbb" size="3">文章部分内容参考自《你不知道的JavaScript（下卷）》及阮一峰《[ECMAScript 6 入门](http://es6.ruanyifeng.com/)》</font>*