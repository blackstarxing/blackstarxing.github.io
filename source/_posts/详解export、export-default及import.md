---
title: 详解export、export default及import
date: 2019-04-15 23:31:02
urlname: es6-module
categories: ["技术"]
tags: ["JS","ES6"]
toc: true
---

相比之前的`CommonJS`和`AMD`模块方案在运行时加载，ES6实现了模块的静态化使得编译时就能确定模块的依赖关系、输入以及输出的变量，这样可在编译时选择性加载，效率更高。

ES6模块由两个关键字构成:``import``和`export`。直白点说就是导入和导出。

### export

`export`有两种导出方式:变量/函数声明导出和命名导出。

``` javascript
export var name = 'blackstar'
export function bar(){..}

var name = 'blackstar'
function bar(){..}
export {name, bar}
```

当然你也可以在导出的时候通过`as`关键字对变量进行重命名。

``` javascript
function bar(){..}
export { bar as baz }
```

重命名后的名字放在`as`之后。

`注意:export规定对外导出的变量必须与模块内部的变量一一对应，以下的写法会报错`:

``` javascript
export 1

var a = 1
export a
```

这两种写法其实都是导出了一个值1，并不能对应到模块中的变量，所以`export`导出的必须是个变量而不是值。

通过`export`导出的值是实时动态的，这是因为`export`绑定的是变量本身的引用或者指针，当模块内的变量值发生了改变，引入的变量值也会发生改变。

### export default

尽管可以在模块内部多次使用`export`，但ES6倾向于一个模块使用一个`export`，称为默认导出。`export default`用于指定模块的默认输出，一个模块只能有一个默认输出。

```javascript
export default function () {
  console.log('foo')
}
```

`export default`其实是输出了一个叫做`default`的变量，所以它后面不能跟变量声明语句，但是可以跟函数表达式，不过函数名在模块外部无效，视同匿名函数。`export default`也可以导出一个数值，因为相当于把值赋给`default`变量导出。

```javascript
export default var a = 1 // error 
export default function foo() {
  console.log('foo')
}
export default 42
```

### import

`import`关键字一共有四种加载方式:

1、

``` javascript
import { foo, bar as baz } from './foo'
```

通过这种方式可以导入一个模块API的某些特定命名成员到你的顶层作用域中，并运行模块。这是一种窄导入的思想，假设一个模块一共导出了10个方法或变量，我们在另一个模块中只需要其中2个就可以按需加载，避免不必要的浪费。只是需要注意，导入的变量名字必须和导出的变量名一致。和`export`一样，`import`也可以通过`as`对变量进行重命名。`import`后面的`from`指定模块文件的位置，可以是相对路径，也可以是绝对路径，`.js`后缀可以省略。

2、

``` javascript
import _ from 'lodash'
```

由于`export default`(默认导出)就是输出一个叫做`default`的变量或方法，所以允许你为它取任意名字，又因为一个模块只会有一个`export default`，所以也就可以省略大括号的写法。上面的代码相当于:

``` javascript
import { default as _ } from 'lodash'
```

3、

``` javascript
import 'lodash'
```

该写法表示只运行模块而不引入任何模块中的方法或变量。

4、

窄导入的设计思想允许我们按需引入，但是也会有需求将模块所有变量和方法加载进来，为了避免逐个声明，import提供了一种语法变体叫做命名空间导入。

``` javascript
// util.js
export function foo(){..}
export function bar(){..}

// main.js
import * as util from './util'
util.foo()
util.bar()
```

将整个API导入到一个命名空间。

`注意:所有导入的绑定都是不可变和/或只读的，导入之后所有试图赋值的动作都会抛出错误。`

上面的1、2；2、4也可以结合使用。如果想在一条`import`语句中同时输入默认方法和其他接口，可以写成下面这样。

```javascript
export default function (obj) {}
export function bar() {..}
export function foo() {..}

import _, { bar, foo } from 'lodash'
```

语法要求不带`as`的默认值永远在最前。

最后，需要提醒一下，`import`和`export`命令只能在模块的顶层作用域，不能在代码块之中（比如，在`if`代码块之中，或在函数之中）。

<br>

*<font color="#bbb" size="3">部分内容参考自以下：</font>*
[http://es6.ruanyifeng.com/#docs/module](http://es6.ruanyifeng.com/#docs/module)