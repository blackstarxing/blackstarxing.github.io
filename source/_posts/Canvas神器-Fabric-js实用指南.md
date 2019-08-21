---

title: 'Canvas神器:Fabric.js实用指南'
date: 2019-08-17 22:13:36
urlname: fabric-doc
categories: ["技术"]
tags: ["JS","Canvas"]
toc: true
---

`Fabric.js`是一个功能强大且简单的`HTML5`画布库，它给`canvas`上的元素提供了交互式对象模型。借助`Fabric.js`，你可以轻松地绘制各种图形线条图片，对它们进行拖拽、旋转、形变等操作，并且支持丰富的事件方法。

### 绘制图形

所有的绘制开始之前你需要先创建画布对象:

``` html
<canvas id="canvas" width="600" height="600"></canvas>
```

``` javascript
var canvas = new fabric.Canvas('canvas')
```

#### 绘制圆形

```javascript
const circle = new fabric.Circle({
  radius: 100,
  left: 10,
  top: 10,
  fill: 'yellow',
  strokeWidth: 2,
  stroke: "red"
})
canvas.add(circle)
```

| 属性        | 作用                       |
| ----------- | -------------------------- |
| radius      | 半径大小                   |
| left        | 以画布左上角为顶点的横坐标 |
| top         | 以画布左上角为顶点的纵坐标 |
| fill        | 对象的填充颜色             |
| strokeWidth | 外边框线宽度               |
| stroke      | 边框颜色                   |

![circle](https://s2.ax1x.com/2019/08/18/mQZMrV.png)

#### 绘制矩形

``` javascript
const rect = new fabric.Rect({
  width: 200, 
  height: 100, 
  rx: 20, // 圆角半径（x轴方向）
  ry: 20, // 圆角半径（y轴方向）
  fill: '#FF4D4F',
  strokeWidth: 2,
  stroke: "#dedede"
})
canvas.add(rect)
```

![rect](https://s2.ax1x.com/2019/08/18/mlEO2R.png)

利用`rx`、`ry`这两个圆角属性你也可以通过`fabric.Rect`绘制圆形和椭圆形。

#### 绘制三角形

```javascript
const triangle = new fabric.Triangle({
  width: 200, 
  height: 300, 
  fill: '#FF4D4F',
  strokeWidth: 2,
  stroke: "#dedede"
})
canvas.add(triangle)
```

![triangle](https://s2.ax1x.com/2019/08/18/mlEvKx.png)

### 画线

```javascript
const canvas = new fabric.Canvas('c')
const grid = 30
const lineStroke = '#C5C9CB'
for (let i = 0; i < (canvas.width / grid); i++) {
  const lineX = new fabric.Line([ 0, i * grid, canvas.width, i * grid], {
    stroke: lineStroke,
    selectable: false,
    type: 'line'
  })
  const lineY = new fabric.Line([ i * grid, 0, i * grid, canvas.width], {
    stroke: lineStroke,
    selectable: false,
    type: 'line'
  })
  canvas.add(lineX)
  canvas.add(lineY)
}
```

`fabric.Line([50, 100, 200, 200]`的四个数值分别对应线起点和终点的横纵坐标，利用`Line`类我们可以实现网格线的绘制。`selectable`属性值为`false`的时候表示对象不可选中，`type`属性可以自定义对象类型。

![line](https://s2.ax1x.com/2019/08/19/m8lGDg.png)

### 添加文字

``` javascript
const text = new fabric.IText('blackstar', {
  top: 100,
  left: 100,
  fontFamily: 'Calibri',
  fontSize: 18,
  angle: 45,
  originX: 'center',
  originY: 'center',
  centeredRotation: true,
  fill: '#000'
})
canvas.add(text)
```

| 属性             | 作用                                             |
| ---------------- | ------------------------------------------------ |
| fontFamily       | 字体                                             |
| fontSize         | 文字大小                                         |
| angle            | 角度                                             |
| originX          | X轴中心点位置                                    |
| originY          | Y轴中心点位置                                    |
| centeredRotation | 如果为true，则此对象将使用中心点作为旋转时的原点 |

注意文字内容必须为`字符串`类型，否则会报错。

![text](https://s2.ax1x.com/2019/08/18/mleuCQ.png)

### 绘制图片

``` javascript
fabric.Image.fromURL('https://s2.ax1x.com/2019/08/18/mlm0Jg.jpg', function(oImg) {
  oImg.scale(0.5)
  canvas.add(oImg)
})
```

![image](https://s2.ax1x.com/2019/08/18/mlmjYD.png)`scale()`方法用于控制对象的缩放。

### 群组绘图

群组是`Fabric.js`最强大的功能之一。试想一下，如果你需要在一个方框中添加一些文字你该怎么做？先添加一个矩形，再把文字覆盖在上面吗？`Fabric.js`提供了`Group`类帮助我们将多个对象组成一个对象。像上面提到的需求我们就可以这样来实现:

``` javascript
const rect = new fabric.Rect({
  width: 200, 
  height: 100, 
  rx: 20,
  ry: 20,
  fill: '#FF4D4F'
})
const text = new fabric.IText('blackstar', {
  fontFamily: 'Calibri',
  fontSize: 18,
  left: rect.width/2,
  top: rect.height/2,
  originX: 'center',
  originY: 'center',
  centeredRotation: true,
  fill: '#fff'
})
const block = new fabric.Group([rect,text], {
  left: 50,
  top: 50
})
canvas.add(block)
```

![group](https://s2.ax1x.com/2019/08/19/m8t7lR.png)

`Group`类接受对象数组将其组装在一个对象中，排在后面的绘制在上层。

#### 文字垂直水平居中

在群组中设置文字垂直水平居中其实很简单，只需要将文字的坐标中心点通过`originX`、`originY`设置在正中央，横纵坐标分别设置为包裹区域宽高的一半就可以了。

### 对象属性汇总

| 属性            | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| left            | 横坐标                                                       |
| top             | 纵坐标                                                       |
| width           | 宽度                                                         |
| height          | 高度                                                         |
| originX         | 对象转换的水平原点('left'，'right'，'center')                |
| originY         | 对象转换的垂直原点('left'，'right'，'center')                |
| angle           | 偏转角度                                                     |
| snapAngle       | 设置对象在旋转时锁定的角度。比如设置值为45则对象在旋转时只能以45度为一个单位 |
| fill            | 对象的填充色                                                 |
| backgroundColor | 对象的背景色                                                 |
| hasControls     | 值为false时无法对对象进行旋转和拉伸                          |
| selectable      | 对象是否可选中，false时无法选中                              |
| lockRotation    | true时无法旋转对象                                           |
| lockMovementX   | true时对象无法水平移动                                       |
| lockMovementY   | true时对象无法垂直移动                                       |
| scaleX          | 水平方向缩放倍数                                             |
| scaleY          | 垂直方向缩放倍数                                             |
| stroke          | 线颜色                                                       |
| strokeWidth     | 线宽度                                                       |
| fontFamily      | 字体                                                         |
| fontSize        | 字号                                                         |
| type            | 标记对象类型                                                 |

除了官方文档中给定的属性，你还可以根据需要自定义属性名。

### 对象常用方法

| 方法                     | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| object.set()             | 设置对象属性，如rect.set({top: 50,left:100})                 |
| object.scale()           | 缩放对象                                                     |
| object.rotate()          | 旋转对象                                                     |
| object.getBoundingRect() | 返回对象的边界矩形（左，顶部，宽度，高度）的坐标             |
| object.setCoords()       | 当对象修改了坐标、长宽、缩放、角度、倾斜程度等可能改变对象位置的属性时需要通过该方法重新计算位置 |

### 画布方法

| 方法                         | 作用                                           |
| ---------------------------- | ---------------------------------------------- |
| canvas.setActiveObject(obj)  | 将给定对象设置为画布上唯一的活动对象           |
| canvas.getObjects()          | 获取画布上的所有实例对象                       |
| canvas.add(obj)              | 添加实例对象                                   |
| canvas.remove(obj)           | 移除对象                                       |
| canvas.renderAll()           | 重新渲染画布                                   |
| canvas.sendToBack(o)         | 将对象或多个选择的对象移动到绘制对象堆栈的底部 |
| canvas.discardActiveObject() | 丢弃当前的活动对象                             |
| canvas.clear()               | 清除实例的所有上下文                           |
| canvas.dispose()             | 清除canvas元素并删除所有事件侦听器             |
| canvas.backgroundColor       | 设置画布背景色                                 |

### 事件监听

`Fabric.js`提供了全方位的事件监听机制：

![events](https://s2.ax1x.com/2019/08/21/mtkq2D.png)

常用的有以下几个：

```javascript
// 对象移动监听
canvas.on('object:moving', function(e) {
  console.log(e.target)
})
// 对象缩放监听
canvas.on('object:scaling', function(e) {
  console.log(e.target)
})
// 对象旋转监听
canvas.on('object:rotating', function (e) {
  console.log(e.target)
})
// 对象变动监听，包括位置、大小、角度等变化的监听
canvas.on('object:modified', function(e) {
  console.log(e.target)
})
// 点击事件监听
canvas.on('mouse:down', function (e) {
  console.log(e.target)
})
canvas.on('mouse:up', function (e) {
  console.log(e.target)
})
```

### 边界检测

在操作画布上的实例对象时我们并不希望它们超出画布的范围，那么就需要进行边界判断，控制对象不要越过指定区域。

``` javascript
canvas.on('object:moving', function (e) {
  checkBoudningBox(e)
})
canvas.on('object:rotating', function (e) {
  checkBoudningBox(e)
})
canvas.on('object:scaling', function (e) {
  checkBoudningBox(e)
})
// 此为对象坐标原点设置在中心的情况
function checkBoudningBox(e) {
  const obj = e.target

  if (!obj) {
    return
  }
  obj.setCoords()
  
  const objBoundingBox = obj.getBoundingRect()
  if (objBoundingBox.top < 0) {
    obj.set('top', objBoundingBox.height/2)
    obj.setCoords()
  }
  if (objBoundingBox.left > canvas.width - objBoundingBox.width) {
    obj.set('left', canvas.width - objBoundingBox.width/2)
    obj.setCoords()
  }
  if (objBoundingBox.top > canvas.height - objBoundingBox.height) {
    obj.set('top', canvas.height - objBoundingBox.height/2)
    obj.setCoords()
  }
  if (objBoundingBox.left < 0) {
    obj.set('left', objBoundingBox.width/2)
    obj.setCoords()
  }
}
```

原理就是利用事件监听和对象边界矩形的顶点坐标，当坐标值超出画布范围时重置对象坐标。