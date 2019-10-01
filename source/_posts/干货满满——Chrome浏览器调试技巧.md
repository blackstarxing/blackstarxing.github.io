---
title: 干货满满——Chrome浏览器调试技巧
date: 2019-09-27 21:38:09
urlname: chrome-devtools
categories: ["技术"]
tags: ["浏览器调试"]
toc: true
---

作为一款深受开发者喜爱的浏览器，`Chrome`配备的强大好用的开发者工具`Devtools`功不可没。开发者工具面板包含了：

- `Elements(元素)`面板
- `Console(控制台)`面板
- `Sources(源代码)`面板
- `Network(网络)`面板
- `Performance(性能)`面板 
- `Memory(内存)`面板
- `Application(应用)`面板
- `Security(安全)`面板
- `Audits(审计)`面板

等，覆盖了和开发有关的各个方面。但如果你和曾经的我一样日常仅仅使用`Devtools`来审查网页元素、打印变量和查看网络请求的话显然没有发挥出它的威力，接下来的时间不妨跟着我一起了解学习一下`Chrome`浏览器的调试技巧。

### 快捷键

掌握快捷键可以为你省去滑动鼠标找菜单的时间，提高工作效率，所以务必熟记。

`注：本文所有快捷键针对Mac系统`

#### 快速打开Devtools面板

你可以通过组合键`command+option+i`或`command+option+j`快速打开开发者工具，前者会默认打开上一次关闭时使用的功能面板，后者默认打开`Console`面板。此外还有另一组我最常使用的快捷键:`command+shift+c`，这样默认打开的是`Element`面板，并启用元素审查。

![i](https://s2.ax1x.com/2019/09/29/uGD55j.gif)

#### 切换面板位置

根据不同人的使用习惯、屏幕大小或者面板内容，我们有时需要切换面板显示位置，都知道可以点击三个小点点处改变面板位置:

![dock-side](https://s2.ax1x.com/2019/09/29/uG0GdJ.png)

但你也可以用快捷键`command+shift+d`快速实现:

![toggle-dock](https://s2.ax1x.com/2019/09/29/uGswhd.gif)

#### 切换设备模式

使用快捷键`command+shift+m`切换至设备模式，调试移动端页面:

![device](https://s2.ax1x.com/2019/09/29/uG6cFg.gif)

#### 切换面板

使用`command+[`和`command+]`左右切换功能面板:

![toggle-panel](https://s2.ax1x.com/2019/09/30/utvlqI.gif)

### 查看鼠标悬浮状态下的元素样式

当我们想查看一个元素在鼠标悬浮状态下的样式时会发现无法保持住`hover`状态，要是能有办法将页面定住就好了。你别说，还真有。首先将面板切换至`Source`，把鼠标悬浮到待查看的元素上，按下`command+\`，此时页面和`debug`模式一样，脚本已经暂停运行，这个时候再选中该元素就可以看到其`hover`下的样式了。

![hover-style](https://s2.ax1x.com/2019/10/01/uNZFUg.gif)

另一种方法是在元素面板下，在目标元素上右键唤起菜单，选择`Force state`强制修改状态:

![force-state](https://s2.ax1x.com/2019/10/01/uUVvSs.gif)

### 命令菜单(Command Menu)

`Devtools`功能十分强大，但毕竟面板有限无法同时展现全部功能，你可以通过`command+shift+p`打开命令菜单查看全部功能:

![command-menu](https://s2.ax1x.com/2019/10/01/uUMDhT.gif)

#### 自带截屏功能

`Chrome`自身配备了截图功能，而且非常好用。打开上面的命令菜单输入`screenshoot`可以看到筛选出来了四种截图方式，由上到下分别为自定义区域截图、长图截图、`Dom`节点截图、当前可视区域截图。

![screenshot-menu](https://s2.ax1x.com/2019/10/01/uU17gU.png)

![screenshot](https://s2.ax1x.com/2019/10/01/uU1cjg.gif)

### 控制台小技巧

诚然使用`console.log`足够完成一般的调试工作，但掌握更多的控制台技巧则会为你的调试添砖加瓦。

#### 将变量名一起打印

只打印变量值的话当日志内容多起来的时候很容易分不清是谁的值，应该将变量用大括号括起来打印键值对:

![console-object](https://s2.ax1x.com/2019/10/01/uU21UI.png)

#### 打印对象数组

对于数组对象，打印时可以使用`console.table`让他们更整洁，还可以传入第二个参数控制需要展示的属性，并且支持排序:

![console-table](https://s2.ax1x.com/2019/10/01/uU2vdA.png)

#### 断言打印

使用`console.assert`，当条件为假时打印内容:

![console assert](https://s2.ax1x.com/2019/10/01/uUIs0S.png)

#### 打印上一次运行结果

使用`$_`打印上一次运行结果:

![$_](https://s2.ax1x.com/2019/10/01/uUIckQ.png)

#### 获取当前选中节点以及之前选中节点

`$0`可以获取当前选中节点，`$1`~`$4`获取最近选中的4个节点:

![$0](https://s2.ax1x.com/2019/10/01/uUIyTg.png)

#### 将DOM结点以DOM树的结构进行输出

使用`console.dir`可以将`DOM`结点以`DOM`树的结构进行输出，方便查看属性方法:

![console dir](https://s2.ax1x.com/2019/10/01/uUoVjP.png)

#### 为打印结果加时间戳

在命令菜单中找到`Show timestamps`可以为打印结果加上时间戳:

![show timestamps](https://s2.ax1x.com/2019/10/01/uU4HDU.gif)

#### copy

`copy`方法可以将内容复制到系统的剪贴板中，并返回格式化后的结果:

![copy](https://s2.ax1x.com/2019/10/01/uUoNHU.gif)

### 代码片段

`Devtools`为我们提供了一个功能用来快速运行一段`JavaScript`代码，并且可以保存下来在各个页面复用，这意味你无需反复的编写相同的代码，也不用为了验证一段代码的结果而新建一个文件。

它就是位于`Source`面板的`Snippets`:

![snippets](https://s2.ax1x.com/2019/10/01/uUTjeK.gif)

上图我创建了一个代码片段并声明了一个时间戳格式化的方法，保存后右键文件或`command+Enter`运行代码了，此时该方法已被创建并可使用。

### 改变网络环境

有些时候我们需要模拟不同的网络情况进行弱网测试，尤其是针对移动端，那么如何在浏览器上面做到？在`Network`面板上就有关于这种需求的设置，有几个预设网络情况，你还可以新增模式进行具体的限速:

![slow net](https://s2.ax1x.com/2019/10/02/uU7gte.gif)