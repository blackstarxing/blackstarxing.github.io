---
title: 编写优雅的JavaScript代码
date: 2019-01-10 22:28:35
urlname: simple-your-code
categories: ["技术"]
tags: ["JS"]
toc: true
---

在阅读别人的代码时，你一定有过不少大呼竟然还有这种操作的时刻，一段相同的逻辑或功能，为什么别人只用5行就搞定，而自己却用了10行甚至更多。冗余复杂的代码不仅可读性差，效率也可能受到影响，那么在JavaScript中都有哪些小技巧来让我们的代码更加优雅呢？

### 优化条件判断

条件判断其实是最能区分小白和有经验工程师的标准之一，如果一段代码频频出现`if else`，想必看的人一定是眉头紧锁的。其实对于条件判断有着很多种优化的方法。

#### 逻辑运算符

``` javascript
if (sex === 'male') {
    if (age>18) {
        // ...
    }
}

if (age < 18) {
    // 公交半价 
} else if(age > 60) {
    // 公交半价
}
```

看上面两段代码，脑壳疼不，对于刚开始写代码的新人来说这种情况一定不少。我们用`&&`以及`||`一句话实现上述逻辑：

``` javascript
if (sex === 'male' && age > 18) {
    // ...
}

if (age < 18 || age > 60) {
    // 公交半价
}
```

#### 三元表达式

对于非真既假的条件判断往往比较适合使用三元表达式来替代`if else`：

``` javascript
let sex = user.sex === 0 ? '男' : '女'
```

#### switch case

对于状态值比较多的条件判断，使用`switch case`会比一排排使用`if {} else if {} else if {}`更加合理：

``` javascript
if (state == '1') {
    // do something
} else if (state == '2') {
    // do something
} else if (state == '3') {
    // do something
} // ...

// ❌ 

switch (state)
{
    case 1:
        //do something
        break;
    case 2:
        //do something
        break;
    case 3:
        //do something
        break;
}
```

#### 利用对象结构判断状态

``` javascript
let state = {
    1: 'success',
    2: 'error',
    3: 'unknown'
}
function foo(code) {
    return state[code]
}
```

#### 短路求值

短路求值的意思是说只有当第一个运算数的值无法确定逻辑运算的结果时，才对第二个运算数进行求值：当`&&`的第一个运算数的值为`false`时，其结果必定为`false`，无需再运行第二个运算数，除非第一个运算数结果为`true`；`||`也一样。

``` javascript
// 普通的if语句
if(test){
    isTrue()  // Test is true
}

// 上面的语句可以使用 '&&' 写为：

( test && isTrue() )  // Test is true
```

而`||`可以给参数设置默认值：

``` javascript
function (name) {
    let name = name || 'Blackstar'
}
```

#### every()、some()

判断数组中是否全部或部分满足条件时可以使用`every()`或`some()`方法：

```  javascript
let arr = [1,2,3,4,5]
arr.every((item) => {
    return item > 3  // false
})
arr.some((item) => {
    return item > 3  // true
})
```

#### 减少嵌套，提早return

当条件判断逐级递进时，一层层嵌套堆叠容易做无用功。我们应当遵循当发现无效条件时及时`return`的规则。

``` javascript
function test(fruit, quantity) {
    // 条件 1：fruit 必须有值
    if (fruit) {
        // 条件 2：必须为苹果
        if (fruit == 'apple') {
            console.log('red')
            // 条件 3：数量必须大于 10
            if (quantity > 10) {
                console.log('big quantity')
            }
        }
    } else {
        throw new Error('No fruit!')
    }
}
 
// 测试结果
test(null) // 抛出错误：No fruits
test('apple') // 打印：red
test('apple', 20) // 打印：red，big quantity
```

这一层层的判断像极了回调地狱，而我们要看到最下面才会知道非水果的判断逻辑，而使用提早`return`的规则：

``` javascript
function test(fruit, quantity) {
    // 条件 1：提前抛出错误
    if (!fruit) throw new Error('No fruit!')
    // 条件2：必须为苹果
    if (fruit == 'apple') {
        console.log('red')
        // 条件 3：数量必须大于 10
        if (quantity > 10) {
            console.log('big quantity')
        }
    }
}
```

及早的抛出错误，减少嵌套，保持良好的编码风格，无疑提高了程序的可读性。

### 善用ES6

ES6前面的文章有介绍过（[ES6——一场JavaScript语法的重大变革](/2018-11-28-es6-change.html)），除了弥补了一些设计缺陷，最主要的还是帮助我们简化了代码。

#### 箭头函数

箭头函数不必多说：

``` javascript
// ES5
function foo(x,y) { 
    return x + y
} 
// ES6
var foo = (x,y) => x + y
```

#### 参数默认值

对上面短路求值的进一步优化

``` javascript
function (name = 'Blackstar') {
	// ...
}
```

#### includes()多条件判断

判断变量是否满足其一结果时，不停使用`||`运算？使用`some()`方法？No，在ES6中，我们只需这样使用includes()方法：

``` javascript
function test(fruit) {
    // 条件提取到数组中
    const redFruits = ['apple', 'strawberry', 'cherry', 'cranberries']
    if (redFruits.includes(fruit)) {
        console.log('red')
    }
}
```

#### 二维及多维数组展开

借用`...`扩展运算符及`concat()`方法，`concat()`用于连接两个或多个数组。

``` javascript
// 二维数组
let a = [1,2,[3,4]]
a = [].concat(...a)
// 多维数组
function flattenArray(arr){
    const flattened =[].concat(...arr)
    return flattened.some(item =>Array.isArray(item))?flattenArray(flattened):flattened
}
const arr =[11,[22,33],[44,[55,66,[77,[88]],99]]]
const flatArr = flattenArray(arr) // => [11, 22, 33, 44, 55, 66, 77, 88, 99]
```

#### 数组去重

``` javascript
let numList = [1,2,3,3,4,5]
Array.from(new Set(numList)) // [1,2,3,4,5]
```

`Set`保证了数据结构中成员的值是唯一的，而`Array.from`可以将这种数据结构转变为真正的数组。

#### 异步处理

避免回调地狱，请使用`Promise`，参看前文（[JS单线程与异步](/2018-09-05-javascript-singlethread-asynchronous.html)），不再赘述。

*<font color="#bbb" size="3">部分内容参考自以下：</font>*
https://www.css88.com/archives/9865