---
title: React受控组件与非受控组件
date: 2020-04-05 19:16:26
urlname: react-controlled&uncontrolled-components
categories: ["技术"]
tags: ["React"]
---

### 受控组件

我们在React中使用表单元素如`input`时，为了方便获得表单元素的属性值通常会给元素绑定状态(state)，并根据用户输入通过`onChange`事件来更新(且这种更新只能通过`setState()`进行)数据。像这种state为唯一数据源，且由React控制着输入操作的表单输入元素就叫做`受控组件`。

``` jsx
class Form extends React.Component {
  constructor(props){
    super(props);
    this.state = {
      name: ''
    }
  }
  handleSubmitClick() {
    console.log(this.state.name)
  };
  handleInput(event) {
    this.setState({
      name:event.target.value
    })
  }
  render() {
    return (
      <div>
        <input type="text" value={this.state.name} onChange={(e) => this.handleInput(e)} />
        <button onClick={() => this.handleSubmitClick()}>Search</button>
      </div>
    );
  }
}
ReactDOM.render(<Form />, document.getElementById("root"));
```

### 潜在问题

但由于表单的属性值完全由React控制，如果值的更新发生在异步操作中就会出现异常情况。看下面这个动图，我用了`setTimeout`模拟异步更新状态的过程，在输入框中快速地输入`123`，最后输入框中却只留下了`3`。

![asyc-controlled](https://s1.ax1x.com/2020/04/04/G0D759.gif)

因为异步更新会和输入过程产生交错，从而吞掉部分输入，异步过程越长影响就越明显，反之则影响越小。但这只是针对输入数字和字母，如果输入中文问题就更大了。

这一次我把`setTimeout`的延迟时间设置为`0`以便让更新状态的任务马上进入任务队列，尽可能早的执行:

![asyc-IME](https://s1.ax1x.com/2020/04/04/G0TNw9.gif)

输入一串英文可以看到没什么问题，输入框中显示的是预期值，但当输入中文时却发现打出的是一串乱掉的拼音，根本没有办法打出文字。这是因为中文输入法需要我们在确定输入前选定结果，然而异步更新却在时刻改变着值，输入法无法正常工作，结果也就变得混乱。

**(多说一句，之所以发现这个问题是因为我在使用Ant Design Mobile的`InputItem`组件时出现了相同的问题，说明`InputItem`的onChange回调也是个异步操作。)**

一切都是因为React时刻控制着表单数据，要想屏蔽以上的bug我们需要寻找到另一种方案来使表单数据脱离React控制的同时还能保证可以随时取得我们想要的数据。

### 非受控组件

>  `Refs`提供了一种方式，允许我们访问`DOM`节点或在`render`方法中创建的`React`元素。

通过使用`ref`来从`DOM`节点中获取表单数据，我们可以编写一个非受控组件。非受控组件将真实数据储存在`DOM`节点中，数据的获取来自于`DOM`节点，`React`无法影响它，所以即便触发了异步操作也不用有任何担心。

代码以及实际效果:

``` jsx
class Form extends React.Component {
  constructor(props){
    super(props);
    this.state = {
      name: ''
    }
  }
  handleSubmitClick() {
    const name = this._name.value;
    console.log(this.state.name)
  };
  handleInput(e) {
    const $this = this
    const name = event.target.value;
    setTimeout(()=>{
      $this.setState({
        name
      })
    },1000)
  }
  render() {
    return (
      <div>
        <input type="text" ref={(input) => this._name = input} onChange={e=>this.handleInput(e)}/>
        <button onClick={()=>this.handleSubmitClick()}>Search</button>
      </div>
    );
  }
}
ReactDOM.render(<Form />, document.getElementById("root"));
```

![uncontrolled-input](https://s1.ax1x.com/2020/04/04/G0Ob0U.gif)

通过`ref`将表单`DOM`赋值给一个变量，后续任何时候都可以通过这个变量拿到表单的数据，当然你可以继续将数据存放进`state`，只不过它再也不会影响正常的输入了，也没有办法动态地改变表单数据。

由于非受控组件不使用`value`传值，要设置初始值可以通过`defaultValue`:

```html
<input defaultValue="Blackstar" type="text" ref={(input) => this._name = input} />
```

### 怎么选择？

介绍完了受控以及非受控组件，我们在实际的应用中应该如何选择？

我是这样理解的:如果你需要时刻掌握输入的数据并对状态进行控制，比如你需要校验一些输入格式或者是转换大小写，同时场景中既不考虑中文输入也几乎没什么异步过程的话可以放心使用受控组件。

而如果你只是在需要的时候能拿到数据就行，非受控组件显然可以保证不会给你带来什么意外惊喜。