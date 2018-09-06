---
title: JS单线程与异步
date: 2018-09-05 21:09:09
urlname: javascript-singlethread-asynchronous
categories: ["技术"]
tags: ["JS"]
toc: true
---

### 单线程的JS

作为JavaScript的核心特性，单线程决定了其同一时间只能做一件事，所有任务排队等候执行，前一个结束后一个才可以执行。但是当遇到http请求这类比较耗时的任务时，如果持续等待结果将会阻塞整个队列的快速执行，甚至造成浏览器停止响应。举个例子：傍晚时候，我给老王发了条微信，老铁周末去打球吗，老王不一定秒回，如果我一直抱着手机等待老王的回复，那不知道我什么时候可以去吃晚饭，也许刚巧老王前一天通宵到下午才回到家此时正在呼呼大睡，于是在饥肠辘辘的等待中，博主卒。但是如果我发好了微信就去吃饭，什么时候老王回我了再去订球馆就会避免这种情况的出现。好在作为JS宿主环境的浏览器并不是单线程的。

### 多线程的浏览器

浏览器只会给JS分配一个主线程来执行任务，但是浏览器却为那些耗时的任务单独开辟了其他线程，比如：浏览器事件触发线程 、http请求线程 、定时器触发线程 ······当主线程执行到这类耗时任务时便会暂时将它们挂起，继续向下执行后面的任务，等到挂起的任务执行完成后再回过头来进入到主线程继续执行。

### “现在”与“将来”

如此一来，任务被分成了两种，一种是`同步任务`，一种是`异步任务`，同步任务在主线程上执行，主线程之外存在一个`任务队列`，当异步任务有了执行结果就在任务队列当中放置一个事件，等待主线程上的同步任务全部执行完成后便会进入主线程中执行。异步任务都有一个或多个回调函数，当任务队列中的事件被主线程读取时便会执行这个回调函数。在**《你不知道的JavaScript（中卷）》**一书中，作者将其称为“现在”与“将来”。

### 回调函数  

> 回调函数就是一个通过函数指针调用的函数。如果你把函数的指针（地址）作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，我们就说这是回调函数。回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。

可见，回调函数的执行时机要依情况而定。前面说了，异步任务都有一个或多个回调函数，但回调不代表一定是异步操作。比如：

```javascript
function f1(callback){
    console.log(1)
    callback()
}
function f2(){
    console.log(2)
}
f1(f2)
```

这其实是一个同步的回调，因为没有任务发生在“将来”。

而假如f1是一个ajax请求，f2的执行需要依赖于f1返回的数据：

```javascript
function f1(callback){
    // 这里使用setTimeout来模拟ajax请求的耗时过程
    setTimeout(function () {
        // f1的任务代码
        callback()
    }, 1000)
}
function f2(){
    console.log(2)
}
f1(f2)
```

这就变成了一个异步的回调，因为回调的执行发生在“将来”。

回调函数是异步编程最基本的方法，其优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合，流程会很混乱。当回调不断嵌套，便会形成人们熟知的`回调地狱`。

例如：

```javascript
listen("click",function handler(evt) {
    setTimeout(function request() {
        ajax("http://some.url.1",
            function response(text) {
                if (text == "hello") {
                    handler();
                } else if (text == "world") {
                    request();
                }
            });
        },
    500);
});
```



### 事件循环（Event Loop）

介绍完回调函数我们再回过头来看看主线程提取任务队列事件执行的机制。

这里引用一张经典的图示来说明这个过程。

![eventloop](http://oerh3364g.bkt.clouddn.com/eventloop.png)

`stack`代表主线程中执行的同步任务，`WebAPIs`代表浏览器执行异步任务的线程，当异步任务有结果时便会在`callback queue`（也就是上面提到的`任务队列`）中放置一个事件，这个事件并不能立即执行，而是要等主线程上的任务全部结束，`stack`中的任务每执行完一个便会出栈直至栈空。`stack`在忙活了一阵终于把自己手头的工作做完了，然而它并不能马上休息，要先去看任务队列中是否有事件等待，如果有便要去任务队列中提取事件进入自己内部执行，任务队列中的事件遵循先进先出原则。每一次事件循环称为tick，整个过程不断循环，我们将这种运行机制称为`Event Loop（事件循环）`。

有了对于事件循环的理解，我们来看两个出镜率很高的面试题：

1、

```javascript
setTimeout(function(){
    console.log(1)
},0)
console.log(2)
```

`setTimeout`是浏览器window对象的方法，表示延迟执行一个事件，可以接受两个参数，第一个是一个回调方法，第二个是执行代码前需等待的毫秒数。这是一个非常常用的异步方法。

这个例子我们一看，哎呦，第二个参数是0，太好了不用等马上可以执行。于是快速的得出结果：`1 2`。

可`setTimeout`是一个异步任务，当主线程执行到它时会把它交到浏览器单独为其提供的线程中去然后继续向下运行。虽然等待时间为0也只是意味着回调函数可以马上进入到任务队列中等待，直到主线程任务全部结束再通过事件循环机制读取任务队列执行事件。所以正确结果应该是`2 1`。

2、

```javascript
for(var i=0;i<5;i++){
    setTimeout(function(){
        console.log(i)
    },1000)
}
```

这段代码初学者通常会给出`0 1 2 3 4`的回答，但其实`for`循环中`i`的累加过程是同步的，循环体中的打印是异步的，并且`i`是全局变量，当同步任务执行完的时候运行环境中变量`i`的值已经变成了`5`，这个时候再执行定时器回调函数将会连续打印出`5 5 5 5 5 `。

值得注意的是，**HTML5标准规定了setTimeout()的第二个参数的最小值不得低于4毫秒**，因此即使设置了0，也要至少等待4毫秒，而且还要看同步任务是否全部结束以及当前事件是否排在任务队列的第一个，因此这个等待时间并不一定准确。

### Promise

前面说了使用回调来表达程序异步会造成流程混乱，也不利于代码的阅读和维护，如果能让这一过程更加简洁清晰一些就好了。值得开发者们欣喜的是，ES6为我们提供了一个名为`Promise`的范式，可以不把回调事先传给异步任务，而是给我们提供了解其任务何时结束的能力，然后由我们自己的代码来决定下一步做什么。话不多说，我们先来看`Promise`是怎么使用的：

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

`Promise`是一个对象，有以下两个特点。

（1）对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

`Promise`对象是一个构造函数，用来生成`Promise`实例，接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。当异步操作成功或者失败时，`resolve`和`reject`会作为参数传递出去。

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数，`rejected`非必传。

回到上面那段代码，当我们在`Promise`对象中执行一个异步操作时并不需要在内部处理成功或失败的结果，而是把这个状态抛出来在外面链式地进行处理。也就是`Promise`内部不关心如何处理结果，只在完成的将来处理。这样就把执行代码和处理结果的代码清晰地分离了。

而当我们再一次面对多个存在依赖关系执行的异步任务时，也不需要像以往一样横向编程，层层嵌套形成一个`回调地狱`。`Promise`允许我们进行链式编程：

```javascript
function ajax(url, data) {
    let request = new XMLHttpRequest();
    return new Promise(function (resolve, reject) {
        request.onreadystatechange = function () {
            if (request.readyState === 4) {
                if (request.status === 200) {
                    resolve(request.responseText);
                } else {
                    reject(request.status);
                }
            }
        };
        request.open("GET", url);
        request.send(data);
    });
}
ajax("http://...").then(function(req) {
    return ajax(res.url)
}).then(function(value) {
    // success
}, function(error) {
    // failure
})
```

使用链式的`then` 指定一组按照次序调用的回调函数。前一个回调函数，返回的还是一个`Promise`对象，这时后一个回调函数就会等待该`Promise`对象的状态发生变化，才会被调用。这样既保证了任务的有序性，逻辑也更清晰了呢。

`Promise.prototype.catch`方法是`.then(null, rejection)`的别名，用于指定发生错误时的回调函数。因此

```javascript
ajax("http://...").then(function(req) {
    return ajax(res.url)
}).then(function(value) {
    // success
}, function(error) {
    // failure
})
// 也可写成
ajax("http://...").then(function(req) {
    return ajax(res.url)
}).then(function(value) {
    // success
}).catch(function(error) {
    // failure
})
```

### 新的事件循环

`Promise` 不仅使得异步任务实现方式的定义更加良好，对顺序的保证性更强，也尽可能早的将任务队列中事件的执行提前了。

由此我们需要对前面讲过的任务进行更进一步细化的分类：

> macro-task(宏任务)：包括整体代码script，setTimeout，setInterval
>
> micro-task(微任务)：Promise，process.nextTick

微任务可以理解为是挂在事件循环的每个 tick 之后的一个队列。在事件循环的每个 tick 中，微任务产生的回调不会被添加到任务队列中等待下一轮tick读取，而会进入当前 tick 的主线程末尾，相当于插队。

```javascript
setTimeout(function() {
    console.log('setTimeout');
},0)

new Promise(function(resolve) {
    console.log('promise')
    resolve()
}).then(function() {
    console.log('then') // 插到console.log('console')后面
})

console.log('console')
```

因此上面这段代码的结果是

```javascript
// promise
// console
// then
// setTimeout
```

### 结语

在开始动笔写这篇博客前，我参阅了几篇有关这部分内容的博客，更是反复阅读了阮一峰老师的*[JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)*以及*[Javascript异步编程的4种方法](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)*，但在这过程中其实对文章中的一些解读和概念并不能很好的理解，甚至于感觉有些内容存在矛盾，比如对于异步编程方法中回调函数的举例描述以及事件循环主线程和任务队列关系的语义相反的关系描述，这也引起了我极大的求知欲，不停查找资料以求找到更加准确的定义。也在找寻答案的路途中发现很多有关异步的博客都是几乎照搬阮一峰的博客，并没有引入自己的见解和解读，想说写博客的目的既是一个学习过程的记录，同时也是一个发现并解决问题的契机，与其照搬不如直接加个（转）。希望可以和大家共勉，如果发现我的文章内容描述有偏颇也欢迎一起讨论学习。

*部分内容参考自以下：*
[https://www.cnblogs.com/woodyblog/p/6061671.html](https://www.cnblogs.com/woodyblog/p/6061671.html)
[https://blog.csdn.net/jssy_csu/article/details/78627628](https://blog.csdn.net/jssy_csu/article/details/78627628)
[http://www.ruanyifeng.com/blog/2014/10/event-loop.html](http://www.ruanyifeng.com/blog/2014/10/event-loop.html) 
[http://es6.ruanyifeng.com/#docs/promise](http://es6.ruanyifeng.com/#docs/promise)
《你不知道的JavaScript（中卷）》





