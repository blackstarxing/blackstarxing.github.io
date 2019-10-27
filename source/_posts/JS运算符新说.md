---
title: JS运算符新说
date: 2019-10-25 22:58:27
urlname: js-operators
categories: ["技术"]
tags: ["JS"]
toc: true
---

JavaScript 有如下类型的运算符:

> - 赋值运算符(Assignment operators)
> - 比较运算符(Comparison operators)
> - 算数运算符(Arithmetic operators)
> - 位运算符(Bitwise operators)
> - 逻辑运算符(Logical operators)
> - 字符串运算符(String operators)
> - 条件（三元）运算符(Conditional operator)
> - 逗号运算符(Comma operator)
> - 一元运算符(Unary operators)
> - 关系运算符(Relational operator)

赋值运算符自不必多说了，基于`=`将右边的值赋给左边。

### 算数运算符

最常用的算术运算符，加(`+`)、减(`-`)、乘(`*`)、除(`/`)、取余(`%`)、递增(`++`)、递减(`--`)熟悉的不能再熟悉。递增和递减分为前置和后置，区别在于是将自增(自减)后的值赋值还是先赋值再自增(自减)。

``` javascript
// 后置 
var x = 3;
y = x++; 
// y = 3, x = 4

// 前置
var a = 2;
b = ++a; 
// a = 3, b = 3
```

#### 幂运算符

这里着重提一下幂运算符(`**`)。幂运算符返回第一个操作数做底数，第二个操作数做指数的乘方。即a**b = a<sup>b</sup>。幂运算符是右结合的，这和其他运算符不同:

``` javascript
a ** b ** c = a ** (b ** c)
```

但是需要特别注意，底数前不能紧跟一元运算符(`+/-/~/!/delete/void/typeof`)，比如:

``` javascript
-2 ** 2 //报错
```

因为这会有歧义。如果一定要强制使用负底数，或者打破右结合可以使用括号：

``` javascript
(-2) ** 2
(a ** b) ** c
```

### 逗号运算符

逗号运算符用于对它的每个操作数求值（从左到右），并返回最后一个操作数的值。

``` javascript
var num = (1,2,3,4,5);
console.log(num);  //5

var x = 1
console.log((x++,x)) //2
```

### 比较运算符

比较运算符中最值得关注的就是相等运算符(`==`)和严格相等运算符(`===`)。相等运算符在两侧的值类型不相同会先进行类型转换在比较值，只要值相等即为`true`，而严格相等运算符既要比较值也要比较类型。这里就涉及到了类型转换的问题。

#### 比较规则

> - 字符串和布尔值都转为数字再比较
> - 当两侧的值都是字符串时，不会将其转换为数字进行比较而是会分别比较字符串中字符的Unicode编码
> - undefined 与 null 相等，即 undefined == null
> - 两侧有一个是对象，则先调用valueOf()方法或toString()方法将对象转换为其原始值（一个字符串或数字类型的值），再用结果比较
> - NaN和任何值包括它自己比较，结果都为`false`。

#### [] == []和[] == ![]

当两个操作数都是对象时，JavaScript会比较其内部引用，当且仅当他们的引用指向内存中的相同对象（区域）时才相等，即他们在栈内存中的引用地址相同。

`[]`属于引用类型，即便两个都是空数组，但是由于引用地址不同，所以:

``` javascript
[] == [] //false
// 同理
{} == {} //false
```

而对于`[] == ![]`，根据运算符优先级，`!`的优先级大于`==`，所以先会执行`![]`。`!`可以将变量转换成布尔类型，`null`、`undefined`、`NaN`以及空字符串(`''`)取反都为`true`，其余都为`false`。

故`[] == ![]`相当于`[] == false`相当于`[] == 0`。左侧值作为对象需要调用`toString()`方法，`[].toString() = ''`，所以我们来看一下完整的推导过程:

``` javascript
[] == ![] -> [] == false -> [] == 0 -> '' == 0 -> 0 == 0 -> true
```

但是`{}`的转换结果为`NaN`，`NaN == 0`结果为`false`，所以`{} == !{}`结果为`false`。

#### 总结

``` javascript
1 == 1             //true
'1' == 1           //true
false == 0         //true
true == 2          //false
null == undefined  // true
null == 0          //false
undefined == 0     //false
NaN == 0           //false
NaN == NaN         //false
NaN == !NaN        //true
[] == []           //false
[] == ![]          //true
{} == {}           //false
{} == !{}          //false
```

### 位运算符

位运算符包括按位与(`&`)、按位或(`|`)、按位异或(`^`)、按位非(`~`)、左移(`<<`)、有符号右移(`>>`)和无符号右移(`>>>`)，都涉及到二进制数，使用场景比较少，可自行查阅。这里介绍几个位运算符的妙用。

#### 判断奇偶数

``` javascript
num & 1 === 1 // num 为奇数
num & 1 === 0 // num 为偶数
```

因为二进制的奇数最低位是1，偶数最低位是0，按位与任意一个为0结果即为0，否则为1，所以可以判断奇偶数。

#### 取整

``` javascript
5.21 >> 0 // 5
5.21 << 0 // 5
5.21 | 0  // 5
~~ 5.21   // 5
```

但是不可对负数进行取整。

#### 交换数字值

``` javascript
var a = 6
var b = 77
a ^= b
b ^= a
a ^= b
console.log(a)   // 77
console.log(b)   // 6
```

### 运算符优先级

圆括号运算符(`()`)可改变运算符顺序，优先级最高。完整的运算符优先级顺序可查看[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)。