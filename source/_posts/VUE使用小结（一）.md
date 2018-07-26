---
title: VUE使用小结（一）
date: 2018-07-14 22:23:26
categories: ["生活"]
tags: ["前端", "vue"]
toc: true
---

vue作为当下最火的框架之一，上手快社区活跃文档清晰各类插件繁多的特点让其在大量前端团队中备受青睐。

从16年中开始使用vue1.0到逐渐过渡到现在的2.0，两年的时间里vue一直是我团队中主要的前端框架，但在过往的项目中仅仅只是大量的利用了vue单页应用、路由、双向数据绑定以及数据驱动视图的特点，没有最大化的运用vue的功能，想来也是可惜。

时过境迁，公司目前项目的业务特点再加上有前人的技术沉淀，对于vue的开发使用比以往丰富了很多，自身也有意的想更多的去发掘vue的用法，在使用学习的过程中，不断有新的收获，累积总结于此，便开启了vue系列的第一篇。
## 组件通信
当项目中的某一部分功能经常会被复用时，每次都写一段一样的代码显然并不妥当，为了方便我们可以把这部分代码封装成一个组件，当有需要用到的时候直接引用这个组件就好了。那么如何在调用组件的时候向其传入数据以及组件中的行为如何影响到调用他的父组件，简而言之，组件之间的通信是我们需要解决的问题。而组件通信分为父子组件通信和非父子组件通信。

### 1、父子组件通信
#### 父向子传值
``` html
<addScale :title="title"></addScale>
```
向子组件传值的方式很简单，只要通过v-bind绑定特性就可以了，简写就是:title，这样就可以向子组件中传递一个名为title的变量。

``` html
<div>{{title}}</div>
```
``` javascript
export default {
  props: {
    'title': String
  }
}
```

那么在子组件中通过props声明可被接受的数据就可以在子组件中使用了，如上就是在子组件中接收一个类型为字符串的变量，如果不需要校验类型那么直接可以以数组的方式接收。

``` javascript
export default {
  props: ['title']
}
```
如果变量还有声明默认值还有第三种写法：

``` javascript
export default {
  props: {
    title:{
      type: String,
      default: 'hello' 
    }
  }
}
```
#### 子向父发送事件
既然获取数据已经实现了，那么如何在子组件中向父组件发送事件呢，只需通过emit和on就可以实现了。

``` javascript
//子组件
this.$emit('getId',this.id) 
```
``` html
// 父组件
<addScale :title="title" v-on:getId="getId(id)"></addScale>
```
以上，子组件通过emit发送了一个名为getId的事件，并且传递了一个id作为参数，在父组件中我们使用v-on（也可用@getId）来监听这个事件以实现我们想要的功能。
#### 好用的ref
有些时候我们也会有需要在父组件中调用子组件的方法或数据，这个时候ref的优势便体现出来了：

``` html
//父组件
<addGoods ref="goodItem"></addGoods>
```
``` javascript
this.$refs['goodItem'].resetSearch()
```
当在组件上使用ref特性时，ref便指向了这个组件的实例,可以通过这种方法调用子组件上的resetSearch方法，当然也可以使用组件实例中的数据。
而当ref用在普通的DOM元素上时便是指向这个DOM，可以利用它完成一些DOM操作。

#### $parent
$parent可以让子组件调用到父组件的方法,$root指向根节点。
### 2、非父子组件通信

非父子组件通信的核心是拉来一个第三方做中间人，事件的发送和监听都绑定在这个第三方实例上，我们称之为eventbus。使用方法有两种，一种是新建一个bus.js里面new一个Vue实例，在需要使用的组件中引用。

``` javascript
//bus.js
import Vue from 'vue'
export default new Vue
```
``` html
//组件A
<template>
  <div id="componentA">
    <button @click="send">按钮</button>
  </div>
</template> 

import Bus from './bus.js' 
export default { 
  data() {
    return {
      message: 'how are you?'
    }
  },
　methods: {
    send () {
      Bus.$emit('msg', this.message)
    }
  }
}
```
``` javascript
//组件B
import Bus from './bus.js'
export default {
  data() {
    return {
    	message:  ''
    }
  },
  mounted() {
    Bus.$on('msg', (e) => {
　　　console.log(`有人向你打招呼：${e}`)
    })
  }
}
```
这样两个组件都通过中间人Bus来获取和发送事件，便可以实现两个组件间的通信了。

还有一种方法是将eventbus放在vue的根实例下：

``` javascript
new Vue({
  el: '#app',
  router,
  render: h => h(App),
  data: {
    // 空的实例放到根组件下，所有的子组件都能调用
    Bus: new Vue()
  }
})
```
``` html
//组件A
<template>
  <div id="componentA">
    <button @click="send">按钮</button>
  </div>
</template> 

export default { 
  data() {
    return {
      message: 'how are you?'
    }
  },
　methods: {
    send () {
      this.$root.Bus.$emit('msg', this.message)
    }
  }
}
```
这样通过this.$root.Bus去调用eventbus可以不用单独再引入bus.js，所有在根实例下的组件都可以调用。

以上的方法在逻辑简单的组件通信中使用是很方便的，当数据量大逻辑复杂的时候，就可以考虑使用vuex进行状态管理了，关于vuex的相关使用我会找时间单独写一篇博客。