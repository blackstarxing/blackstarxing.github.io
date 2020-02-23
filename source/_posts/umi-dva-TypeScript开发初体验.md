---
title: umi+dva+TypeScript开发初体验
date: 2020-02-22 11:50:56
urlname: umi-dva-typescript
categories: ["技术"]
tags: ["umi","dva","React","TypeScript"]
toc: true
---

在用Angular1.5挣扎开发了一年多以后，我终于在一次公司的组织架构变动后有了机会选用新的技术对已有项目进行重构，并且最终选择了一直想要上手的React作为主要框架。几个星期的摸索探究后，体验颇为良好。

### [umi](https://umijs.org/zh/guide/)

`umi`是一个可插拔的企业级 react 应用框架，功能完善极易上手。它有着非常完备的插件体系，足以应对绝大部分的功能和业务需求，光是官方提供的插件集`umi-plugin-react`就包括了18个常用的进阶功能，像`i18n国际化`、`dva`、`antd`这些都可以在其中进行配置。

`umi`通过`create-umi`提供了脚手架能力，可以根据需要选择是否启用部分功能。项目创建完成后，你可以预览到`umi`的约定目录结构，帮助你快速理解它是怎样工作的。

### TypeScript

`TypeScript`是`JavaScript`的一个超集，为`JavaScript`增加了很多语言特性比如:类型批注和编译时类型检查、类、接口、枚举、模块...等等，扩展了语法。

那我们为什么要用`TypeScript`?

#### 变量声明

以变量为例，我们都知道在`JavaScript`中变量是弱类型，在开发的过程中变量很容易发生类型转换，以至于后来我们根本不知道最开始定义时的目的是创建一个数组、对象、还是字符串？

而在`TypeScript`中我们可以声明变量的类型，任何其他类型的赋值会导致编译报错，这不仅更加安全，而且可读性更强，在团队协作中可以快速让其他成员理解变量的定义。

#### 接口

接口（Interfaces）是`TypeScript`中一个很重要的概念，它是一系列抽象方法的声明，是一些方法特征的集合，这些方法都应该是抽象的，用来定义对象的类型，需要由具体的类去实现，然后第三方就可以通过这组抽象方法调用，让具体的类执行具体的方法。

举个例子，定义一个接口`Person`，包含名字和年龄两个属性，那么再以`Person`为类型去定义一个变量时则必须对应上接口的定义:

``` typescript
interface Person {
  name: string;
  age: number;
}

let jackson: Person = {
  name: 'Jackson',
  age: 50
};

// 多一个属性少一个属性，或者属性的类型对不上都不行

// wrong
let jackson: Person = {
  name: 'Jackson'
};

// wrong
let jackson: Person = {
  name: 'Jackson',
  age: 50,
  gender: 'male'
};

// wrong
let jackson: Person = {
  name: 'Jackson',
  age: '50'
};
```

上述只是简单地介绍了`TypeScript`小小一角的特性，但是已经很明显的看到它可以帮助我们提升代码的质量，所以不妨尝试一下。

### dva

其实在这段期间的开发学习中最让我感到舒爽和提高效率的东西是**`dva`**。

`dva`是一个基于`redux`和`redux-saga`的数据流方案，用以解决`React`中的组件通信和状态管理。`React`自身提供的通信方式需要通过`props`传参并结合事件回传来实现，而要实现数据流则需要引入多个库。

`dva`完美解决了这两个问题并且`API`更为简化，一个简单的数据流向图就能让你快速理解工作原理:

![dva-dataflow](https://s2.ax1x.com/2020/02/23/3lhvXq.png)

<center><font color="gray">图源来自dva官网</font></center>
#### 核心概念

- State：一个对象，保存整个应用状态
- View：React 组件构成的视图层
- Action：一个对象，描述事件
- connect 方法：一个函数，绑定 State 到 View
- dispatch 方法：一个函数，发送 Action 到 State

#### Models

`dva`通过`model`的概念把一个领域的模型管理起来，数据的改变发生通常是通过用户交互行为或者浏览器行为（如路由跳转等）触发的，当此类行为会改变数据的时候可以通过`dispatch`发起一个`action`，如果是同步行为会直接通过`Reducers`改变`State`，如果是异步行为（副作用）会先触发`Effects`然后流向`Reducers`最终改变`State`，`Subscription`可以根据当前的时间、服务器的`websocket`连接、键盘输入、路由变化等触发自动订阅数据。

举一个加载商品列表的例子，`model`对象定义:

```typescript
import api from '@/services/product';

export default {
  namespace: 'products',
  state: {
    list: [],
  },
  reducers: {
    // 修改state,所有直接修改数据的操作都一定是通过reducers实现
    save(state, { payload }) {
      return { ...state, ...payload };
    },
  },
  effects: {
    // 通过接口异步操作
    *getProducts({ payload }, { call, put }) {
      const response = yield call(api.deviceProducts);
      // 发出一个Action,类似于dispatch,数据流向reducers
      yield put({
        type: 'save',
        payload: {
          list: response,
        },
      });
    },
  },
  subscriptions: {
    // 订阅数据
    productsInit({ dispatch, history }) {
      return history.listen(({ pathname, query }) => {
        if (['/kitchen/mealConfig', '/meal/mealConfig'].includes(pathname)) {
          // 进入页面立刻发起请求
          dispatch({ type: 'getProducts' });
        }
      });
    },
  },
};

```

#### connect

在`View`层的使用也很简单:

```jsx
import { connect } from 'dva';

interface ProductProps {}
interface ProductState {}

@connect(({ products }) => ({
  products,
}))
export default class Products extends Component<ProductProps, ProductState> {
  constructor(props: any) {
    super(props);
  }
  refresh(value: any) {
    const { dispatch } = this.props;
    dispatch({
      type: 'products/getProducts',
      payload: {},
    });
  }
  render() {
    const { list } = this.props.products;
    const productsList = list.map((item: any, index: any) => (
        <li key={index}>{item.name}</li>
      ));
    return (
      <ul>{productsList}</ul>
    );
  }
}

```

`dva`通过`connect`方法将model中的`state`和`dispatch`方法按照命名空间传给组件的`props`，可以直接通过`props`使用数据，这样即使`state`发生变化也会重新触发渲染。`View`层还可以通过交互直接用`dispatch`发送`Action`来改变数据。

但是需要注意如果在`constructor`中将`props`中获取的`state`数据赋值给了组件的`state`，而在后续的交互中又触发了数据源的变化则需要通过`componentWillReceiveProps`周期函数重新赋值，这样才能保证视图数据的准确。

```typescript
componentWillReceiveProps(nextProps: any) {
  this.setState({
    productList: nextProps.products.list
  });
}
```

#### 注册结构

`model`分两类，一是全局`model`，二是页面`model`。全局`model`存于 `/src/models/` 目录，所有页面都可引用；**页面model不能被其他页面所引用**。目录规范如下:

```
+ src
  + models
    - g.js
  + pages
    + a
      + models
        - a.js
        - b.js
        + ss
          - s.js
      - page.js
    + c
      - model.js
      + d
        + models
          - d.js
        - page.js
      - page.js
```

### 总结

总体下来，这一套**`umi+dva+TypeScript`**打造的兵器十分趁手，尤其是`dva`的引入让数据的流向更加清晰，逻辑和交互的处理也变得更加简单，随着项目的逐渐趋于复杂，它还可以有更令人惊喜的表现。

<br>

*<font color="#bbb" size="3">部分内容参考自以下：</font>*

[UmiJS官网](https://umijs.org/zh/guide/)

[DvaJS官网](https://dvajs.com/guide/)

