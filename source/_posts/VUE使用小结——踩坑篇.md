---
title: VUE使用小结——踩坑篇
date: 2018-07-27 20:00:17
urlname:  vue-damn-problems
categories: ["技术"]
tags: ["前端", "vue"]
toc: true
---

相信不少朋友在一开始上手做VUE的项目前都没有仔细完整的阅读几遍文档及API，往往都是在项目的过程中有针对性的查阅，也因此踩了不少大坑小坑自己挖的坑。实际上初始阶段我们遇到的很多问题都可以轻易地在文档中找到答案。我也踩过不少类似的坑，这些坑在未来也可能会继续增加，我们不怕犯错误，只是不能总是犯相同的错误，也要思考为什么没有避免这些错误。改正过后把这些踩坑经历总结记录于此，也是给自己一点提醒。

### 组件只能有一个根元素

当我们兴致勃勃的建立好一个新的VUE项目并运行起来的时候，可能却并没有看到预期酷炫的页面，相反展现出来的是编译失败的错误提示:
`Component template should contain exactly one root element. If you are using v-if on multiple elements, use v-else-if to chain them instead.`

那么很可能你组件中html的结构是类似于这样的：
``` html
<template>
  <h3>{{ title }}</h3>
  <div v-html="content"></div>
</template>
```

这就违背了vue中组件只能有一个根元素的原则，解决的方法也很简单，只要用一个父元素将这些内容包裹在一起就好了。
``` html
<template>
  <div>
    <h3>{{ title }}</h3>
    <div v-html="content"></div>
  </div> 
</template>
```

### 修改数组视图无法更新

我们都知道vue的特性之一是响应式，也就是当我们去修改vue实例中的数据时，视图也会及时更新。可当我们乐此不疲的沉醉于这个特性带给我们的便捷时却也惊奇的发现当我们修改了数组类型的变量时，视图无论如何也不发生改变，我们怀疑式地去通过vue-devtool工具查看变量的值，甚至丧心病狂的一遍遍打印变量的值都发现：没有错啊，数组确确实实已经发生了改变，可为什么视图没有进行更新呢？

我们到文档中去找找答案。
>由于 JavaScript 的限制，Vue 不能检测以下变动的数组：
当你利用索引直接设置一个项时，例如：vm.items[indexOfItem] = newValue
当你修改数组的长度时，例如：vm.items.length = newLength

这是官方文档中关于数组更新监测的注意事项，也就是说之所以视图没有更新，是我们直接通过索引来改变了数组中的某一项值或者直接通过修改length改变数组的长度，vue不能检测到这种情况下的变动。

当你试图修改数据的某一个值时,正确的做法是通过Vue的`set`方法或者通过调用`splice()`变异方法触发试图更新：
``` vue
vm.$set(vm.items, indexOfItem, newValue)
//或
vm.items.splice(indexOfItem, 1, newValue)
```

而改变数组长度同样使用`splice()`方法：
``` vue
vm.items.splice(newLength)
```

掌握了这一条我们再也不会被视图更新的问题困扰了，是不是很棒呢！

### 双向数据绑定失效

常规来讲当我们在input上使用了`v-model`指令后，输入框的值会与数据进行同步，可当我们给`v-model`添加了`lazy`修饰符后，同步的条件便转为了`change`事件，所以当我们并不希望数据延迟同步的时候，切记不要使用`lazy`修饰符。

（未完待续...)
