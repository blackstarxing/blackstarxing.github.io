---
title: 二进制流图片转base64或blob地址
date: 2018-08-23 20:28:30
urlname: arraybuffer-to-base64orblob
categories: ["技术"]
tags: ["JS"]
---

前不久做了一个在前端展示小程序二维码的需求，怎料后台给出的接口并没有返回图片url地址，而是直接以二进制流的形式返回了一张图片。

### base64编码

我们知道img标签的src属性通常接收的值是一个路径地址，既然没有路径那我们就考虑另外一种方法，即base64编码数据显示图片。

来分析一下我们要做到把二进制流转换成base64编码都需要做哪些事情：

一般情况下后台返回的响应数据类型是json,现在我们需要通过`responseType:arraybuffer`指定二进制流对应的响应类型arraybuffer。

ArrayBuffer本质上是类型化数组，它作为内存区域，可以存放多种类型的数据。但ArrayBuffer不能直接操作，而是要通过类型数组对象或DataView对象来操作，它们会将缓冲区中的数据表示为特定的格式，并通过这些格式来读写缓冲区的内容。这些类型数组对象我们称为“视图”。有以下类型：
>Int8Array：8位有符号整数，长度1个字节。
>Uint8Array：8位无符号整数，长度1个字节。
>Int16Array：16位有符号整数，长度2个字节。
>Uint16Array：16位无符号整数，长度2个字节。
>Int32Array：32位有符号整数，长度4个字节。
>Uint32Array：32位无符号整数，长度4个字节。
>Float32Array：32位浮点数，长度4个字节。
>Float64Array：64位浮点数，长度8个字节。

``` javascript
// 创建一个指向后台返回的二进制流的Uint8视图
new Uint8Array(response.data)
```
这里我们在二进制流上建立一个Unint8的视图来对接收到的二进制流进行操作，将其转化为字符串。这就到了`String.fromCharCode()`方法登场的时间了。

fromCharCode()可接受一个指定的Unicode值，然后返回一个字符串。而其实上面得到的视图是一组字节数组，和一般的数组基本无异。之前有看到别人在处理视图转成字符串用了reduce()方法：

``` javascript
new Uint8Array(response.data).reduce((data, byte) => data + String.fromCharCode(byte), '')
```
利用了reduce()方法累加的作用将视图中的值一个一个的转成字符串然后拼接起来。reduce() 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。
`array.reduce(function(total, currentValue, currentIndex, arr), initialValue)`

| 参数          | 描述   |
| --------     | -----  |
| total        | 必需。初始值, 或者计算结束后的返回值。 |
| currentValue | 必需。当前元素 |
| currentIndex | 可选。当前元素的索引   |
| arr          | 可选。当前元素所属的数组对象  |
| initialValue | 可选。传递给函数的初始值    |

但其实fromCharCode()可以同时接受多个Unicode值，利用ES6的`...`扩展运算符可以将上面获得的类型化数组作为参数序列一次性将它们转化为字符串
``` javascript
String.fromCharCode(...new Uint8Array(response.data))
```
这样是不是优雅多了，现在我们已经得到了由初始的二进制流转化成的字符串，最后只要通过`btoa()`这个方法将字符串转化为base64编码的字符串再拼接在`data:image/jpeg;base64,`后面就可以放入src属性中使用了。

最终的完整处理方式如下：
``` javascript
let imgUrl = 'data:image/jpeg;base64,' + btoa(String.fromCharCode(...new Uint8Array(response.data)))
```
### blob对象

转来转去我们总算是如愿的得到了我们想要的结果，但是在隐约中仿佛想起见过`blob:***`这样的图片地址，研究了一下不禁大拍脑门，之前的实现过程真的是绕了好大的弯，其实只要简简单单的两个语句就可以轻松实现这个需求：
``` javascript
var blob = new Blob([response.data], {type: "image/jpeg"});
var imUrl = URL.createObjectURL(blob);
```
Blob对象表示一个不可变、原始数据的类文件对象,是一个可以存储二进制文件的容器。
``` javascript
var blob = new Blob( array, options );
```
>* array是一个由ArrayBuffer,ArrayBufferView,Blob,DOMString等对象构成的Array，或者其他类似对象的混合体，它将会被放进Blob
>* options提供了一个type属性，代表了将会被放入到blob中的数组内容的MIME类型，默认为`""`

后台给我们返回的数据刚好是ArrayBuffer,不用任何操作直接放进Blob对象，并指定图片MIME类型，我们就创建了一个存放这张图片的Blob对象，再通过`URL.createObjectURL()`方法创建一个指向Blob对象的URL，就可以直接拿来用了。

### 结语

这一次的探究找寻到的两种解决方案虽然难易程度相差比较大，但是过程中学习到的知识点还是很多的。可能如果先知道了第二种方法也就不会花时间去了解第一种方法的实现原理了。所以，相信过程吧，总不会是白忙活的。

*部分内容参考自以下链接：*
[https://www.cnblogs.com/copperhaze/p/6149041.html](https://www.cnblogs.com/copperhaze/p/6149041.html)
[https://stackoverflow.com/questions/9267899/arraybuffer-to-base64-encoded-string](https://stackoverflow.com/questions/9267899/arraybuffer-to-base64-encoded-string)
