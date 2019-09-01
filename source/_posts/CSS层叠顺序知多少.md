---
title: CSS层叠顺序知多少
date: 2019-08-30 22:24:42
urlname: css-stacking
cover: "https://s2.ax1x.com/2019/08/31/mx2R9P.md.jpg"
categories: ["技术"]
tags: ["CSS"]
toc: true
---

### 引言

我们都知道`CSS`中的`z-index`属性可以用来改变定位元素的层叠顺序，但是如果仅仅希望通过不断增大`z-index`数值来达到提高层叠次序的目的就有可能陷入欲求不能的迷惑中，其实关于层叠顺序这背后的学问还真不少。

### 瞧一瞧这是为什么？

我们先来看一组案例：

![table](https://s2.ax1x.com/2019/08/31/mxDgMj.png)

上图所示的两组桌子它们的`dom`结构和定位方式都是一模一样的，但是左边那组1号桌的桌号却显示在了2号桌的上面。

``` html
<style>
  .left,.right{
    position: relative;
    width: 400px;
    height: 200px;
  }
  .table{
    position: absolute;
    top: 25px;
    width: 200px;
    height: 150px;
    border: 3px solid #ddd;
    border-radius: 10px;
    text-align: center;
    line-height: 150px;
    color: #fff;
    font-size: 20px;
  }
  .table1{
    left: 25px;
    background: #7f8bdf;
  }
  .table2{
    left: 175px;
    background: #f59891;
  }
  .table .number{
    position: absolute;
    top: 10px;
    right: 10px;
    width: 30px;
    height: 30px;
    line-height: 30px;
    background: #717477;
    border-radius: 50%;
    z-index: 2;
  }
  .right .table{
    z-index: 1;
  }
</style>

<div class="left">
  左
  <div class="table table1">
    桌
    <div class="number">1</div>
  </div>
  <div class="table table2">
    桌
    <div class="number">2</div>
  </div>
</div>
<div class="right">
  右
  <div class="table table1">
    桌
    <div class="number">1</div>
  </div>
  <div class="table table2">
    桌
    <div class="number">2</div>
  </div>
</div>
```

通过样式代码我们可以看到仅仅是右边的桌子设置了`z-index`数值就造成了这种差异，这是为什么？我们首先欢迎今天的第一位嘉宾——**层叠上下文**登场。

### 层叠上下文

**层叠上下文(stacking context)**，是`HTML`元素的三维概念，这些`HTML`元素在一条假想的相对于面向（电脑屏幕的）视窗或者网页的用户的z轴上延伸，`HTML`元素依据其自身属性按照优先级顺序占用**层叠上下文**的空间。

可以这样理解：我们看到的网页就像是一个桌面，`HTML`元素就是桌面上的一张张纸，**层叠上下文**就像是一个信封可以容纳其它纸张甚至是又一个信封。

![envelope](https://s2.ax1x.com/2019/09/01/nScdCq.md.jpg)

当纸张铺开的时候我们可以一眼看到所有，但是一旦他们堆叠在一起就要有一个谁在上谁在下的顺序。

![paper](https://s2.ax1x.com/2019/09/01/nScOPI.md.jpg)

### 层叠水平

**层叠水平(stacking level)**决定了同一个**层叠上下文**中元素在z轴上的显示顺序，就像桌子上纸张的顺序、信封里信纸的顺序。

### 层叠顺序

#### 规则

**层叠水平**只是一个概念，那么我们根据什么来判定元素的**层叠水平**，有一个著名的示图揭示了规则，这就是**层叠顺序(stacking order)**。在CSS3推出后，这个规则得到了进一步的补充。

![stacking order](https://s2.ax1x.com/2019/09/01/nScBvT.png)

规则很清楚，无需进一步解释说明。

#### 注意事项

**<font color="red" size="3">注意了注意了！</font>**

- **层叠水平**只在同一个**层叠上下文**中有意义，同一**层叠上下文**中谁大谁在上面。我们继续用纸和信封来举例子，如果根据**层叠顺序**，一个信封的**层叠水平**在一张纸的上面，那么信封里的信纸也必然在那张纸的上面，即使信纸的**层叠顺序**非常低，没有办法，容器的级别高，子孙享福。信封里的信纸继续按照规则排序，谁大谁在上面

- 当元素的**层叠水平**完全相同时，处于`dom`流后面的元素会覆盖在上面
- 很多文章说元素只要创建了**层叠上下文**就比普通元素**层叠水平高**

真的是这样吗？我们验证一下就知道了:

``` html
<style>
  .floor{
    width: 300px;
    height: 300px;
    background: #aaabc9;
  }
  .round{
    position: absolute;
    top: 100px;
    left: 100px;
    width: 100px;
    height: 100px;
    text-align: center;
    line-height: 100px;
    border-radius: 50%;
    background: #f59891;
    color: #fff;
    z-index: 1;
  }
  .right .round{
    z-index: -1;
  }
</style>
<div class="left">
  <div class="floor">地板</div>
  <div class="round">
    <div class="number">桌</div>
  </div>
</div>
<div class="right">
  <div class="floor">地板</div>
  <div class="round">
    <div class="number">桌</div>
  </div>
</div>
```

左右两边，地板都是一个普通的块级元素，桌子分别创建了`z-index`为1和-1的**层叠上下文**，可它们并没有全部显示在地板上面，右边桌子显然被地板盖住了，这也符合上面图中负`z-index`在`block`元素下面的规则。

![floor](https://s2.ax1x.com/2019/09/01/nSO64x.png)

因此严谨地说，除了负`z-index`的**层叠上下文**元素，其余**层叠上下文**都比普通元素**层叠水平**高。说了这么多，到底怎么才算创建了**层叠上下文**元素呢，别急，我们接下来就详细说一说。

### 创建层叠上下文

目前一共有三种方式来创建**层叠上下文**：

- 根元素 (`HTML`)自己就是一个**层叠上下文**元素
- **z-index**为数值的定位元素
- `CSS3`的属性(7层层叠顺序图中补充的不依赖`z-index`的**层叠上下文**)

关于定位元素创建**层叠上下文**的方式，如果仅仅将`position`设置成了`absolute`或`relative`而没有设置`z-index`则该元素的`z-index`值为`auto`，仍然是一个普通元素，只是**层叠水平**高些，不会创建**层叠上下文**，只有设置了具体数值哪怕是0才行。至于`fixed`比较特殊，在现代浏览器即便没有设置`z-index`值也会自动创建**层叠上下文**。同一**层叠上下文**中，`z-index`值越大，**层叠水平**越高。

可以创建**层叠上下文**的`CSS3`属性如下：

- `opacity` 属性值小于 1 的元素
- `transform` 属性值不为 "none"的元素
- `mix-blend-mode`属性值不为 "normal"的元素
- `filter`值不为“none”的元素
- `perspective`值不为“none”的元素
- `isolation` 属性被设置为 "isolate"的元素
- 在 `will-change`中指定了任意 CSS 属性，即便你没有直接指定这些属性的值
- `-webkit-overflow-scrolling`属性被设置 "touch"的元素

### 非同辈元素层叠顺序判断

比较两个非同辈元素的层叠顺序时，要先向上找父元素或者祖先元素，如果两个元素互为兄弟节点的祖先元素中有一个创建了**层叠上下文**则这个元素在上，若两个祖先元素都创建了**层叠上下文**则按7层**层叠顺序**和`dom`流顺序判断。如果互为兄弟节点的祖先元素都没有创建**层叠上下文**则所有祖先元素中最先创建非负`z-index`**层叠上下文**的元素在上。

### 总结

回归开头的例子，由于左边的桌子只是设置了绝对定位，并没有创建**层叠上下文**，**当父元素没有创建层叠上下文时，其连同子元素处于同一层叠上下文中，按照7层层叠顺序决定层叠水平**。由于子元素`z-index`值大于0，因此两个桌码的**层叠水平**最高。右边桌子都设置了`z-index`数值，各自创建了一个**层叠上下文**，**层叠水平**相同，后来者居上，**层叠上下文**元素的**层叠水平**会影响子元素，所以2号桌整个桌子都会覆盖在1号桌的上面。

<br>

*<font color="#bbb" size="3">层叠上下文概念摘自[MDN web docs](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context),部分配图来自[Unsplash](https://unsplash.com/)</font>*