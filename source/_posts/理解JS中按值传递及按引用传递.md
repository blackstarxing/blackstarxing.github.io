---
title: 理解JS中按值传递及按引用传递
date: 2019-12-30 22:48:11
urlname: value-copy-and-reference-copy
categories: ["技术"]
tags: ["JS"]
toc: true
---

在程序语言中，变量的赋值以及相互之间的引用绝对是使用最频繁的。但在这种引用的过程中很容易出现一个问题:比如为变量`a`赋了某个值，变量`b`引用了变量`a`，此后我们修改了`b`的内容，却发现有时`a`的值也变了，有的时候却不变，这是为什么？

### 引用指向的是值

**JavaScript**中变量的引用指向的是**值**，即使一个值被多个变量引用，他们所指向的都是同一个值，彼此并无关联。影响引用传递结果的是值的类型。

### 基本类型与引用类型

**JavaScript**值有两大类型:基本类型与引用类型。基本类型包括`数字、字符串、布尔、null、undefined和ES6中的symbol`。引用类型包括`对象（数组和内置对象）和函数`。基本类型值是简单的数据段，直接保存在内存中。引用类型值是对象，可以添加、修改和删除属性或方法，但无法直接访问，需要通过内存地址访问。

当为变量引用基本类型的值时，实际上会拷贝一个值副本给新的变量，引用相同基本类型值的变量相互独立，如下图:

<img src="https://s2.ax1x.com/2020/01/01/lGF1j1.png" alt="value-copy" style="width: 50%!important;display: block;margin: 0 auto;" />

代码验证:

```javascript
// 例一
let num1 = 5
let num2 = num1
num2 ++
console.log(num2) // 6
console.log(num1) // 5
```

可以看到虽然`num2`引用了`num1`的值，但是仅仅是拷贝了相同的值，`num2`的变化不会传递给`num1`。

而当为变量赋予引用类型的值时，拷贝的是内存地址，不同的变量指向的是同一个对象。因此当变量修改了某些属性时会改变其内存地址指向的对象:

<img src="https://s2.ax1x.com/2020/01/01/lGFlcR.png" alt="reference-copy" style="width: 70%!important;display: block;margin: 0 auto;" />

``` javascript
// 例二
let person = {
  name: 'Jackson'
}
let man = person
man.name = 'blackstar'
console.log(person.name) // blackstar

// 例三
let arr1 = [1,2,3]
let arr2 = arr1
arr2.push(4)
console.log(arr2) // [1,2,3,4]
console.log(arr1) // [1,2,3,4]
```

修改变量会改变内存地址指向的对象，由此会影响所有引用同一地址的变量。但前提是修改对象属性，而不是重新赋值新的引用类型值:

``` javascript
// 例四
let person = {
  name: 'Jackson'
}
let man = person
man = {
  name: 'blackstar'
}
console.log(person.name) // Jackson

// 例五
let arr1 = [1,2,3]
let arr2 = arr1
arr2 = [4,5,6]
console.log(arr2) // [4,5,6]
console.log(arr1) // [1,2,3]
```

重新赋值改变了内存地址，指向了一个新的对象，从原本共同引用的对象中脱离开来，自立门户，自然也就不会影响其他变量了。

再来一个组合代码强化一下理解:

``` javascript
// 例六
const obj = {
  info: {
    age: 26
  }
}
let fe = obj.info
let age = obj.info.age
obj.info.age = 16
console.log(fe) // { age: 16 } 引用的对象属性发生变化，变量跟随改变
console.log(age) // 26 变量拷贝的是基本类型值，不会受对象改变影响
fe.age = 41
console.log(obj.info) // { age: 41 } 属性修改，相互影响
fe = {
  name: 'blackstar'
};
console.log(fe); // { name: 'blackstar' }
console.log(obj.info); // { age: 41 } 变量指向了新的对象，不影响原来引用的对象
```

### 函数参数是按值传递？

No!不能这么说，还是和开头我们所说的一样，如何传递仍然取决于值的类型，**`基本类型按值传递，引用类型按引用传递`**。

``` javascript
// 例七
function foo(wrapper) {
  wrapper.a = 42
}
let obj = {
  a: 2
}
foo(obj)
console.log(obj.a) // 42

// 例八
function count(value) {
  value += 10
}
let num = 10
count(num)
console.log(num) // 10
```

你看，我们将一个引用了对象的变量传入函数并且在函数中改变对象的属性，函数外该变量也会同步发生改变，若传入一个基本类型值则不然，这和我们之前实验的结果一致。

### 总结

由于引用类型赋值操作保存的是指向对象的内存地址，指向的是同一对象，因此很容易在传递的过程中相互影响，理解不同类型值的传递规则，明确拷贝的目的是拷贝值副本还是建立联系，这会使数据的流转更加分明和清晰。