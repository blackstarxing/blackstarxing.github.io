---
title: i18next——国际化解决方案
date: 2019-02-18 20:14:17
urlname: i18next
categories: ["技术"]
tags: ["JS", "vue", "Angular", "国际化"]
toc: true
---

### 什么是国际化？

出于国际化的考虑，前端项目需要适应不同的语言和地区，倘若每种语言开发一套页面显然不现实，如果能在不修改内部代码的情况下，根据不同语言及地区显示相应的界面将会非常方便。

而i18n就是顺应这种思路的产物，其来源是英文单词 `internationalization`的首末字符`i`和`n`，18为中间的字符数，是“国际化”的简称。我们将项目中的文案提取为变量并以合乎场景的名字命名，把这些变量对应翻译成各种语言的文案信息保存起来建立不同的语言包，应用时根据语言显示对应语言包的内容。

### i18next

![i18next](https://s2.ax1x.com/2019/02/18/kclsGd.jpg)

[I18next](https://www.i18next.com/)是一个用JavaScript编写的**国际化框架**，提供了标准的i18n功能，并且支持各端应用。前端主流框架像Vue、Angular、React都有很完善且功能强大的插件对应支持，可以说你可以在任何应用中使用i18next来解决国际化。下面我就介绍一下i18next在几种前端框架下的使用方法。

### Vue下使用i18next

好久没有搭建过Vue的项目，现在Vue CLI已经出到3了。而最近发现用npm安装依赖奇慢无比，所以这里改用yarn。

#### 安装yarn

首先安装Homebrew:

``` bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

使用Homebrew安装yarn:

``` bash
brew install yarn
```

#### 新建项目

``` bash
yarn global add @vue/cli
yarn global add @vue/cli-service-global
vue create vue-i18
cd vue-i18
yarn serve
```

#### 安装i18next

有关Vue的i18next插件有好几种，这里我们使用`@panter/vue-i18next`。

安装i18next以及`@panter/vue-i18next`:

``` bash
yarn add i18next
yarn add @panter/vue-i18next
```

#### i18next配置

安装好之后我们到`main.js`中去配置:

``` javascript
import Vue from 'vue'
import App from './App.vue'
import i18next from 'i18next';
import VueI18Next from '@panter/vue-i18next';

Vue.use(VueI18Next);

Vue.config.productionTip = false

i18next.init({    
    lng: 'zh', // 设定语言
    fallbackLng: 'en', // 默认语言包
    resources: {
        //语言包资源
        en: { translation: require('./locales/en.json') },
        zh: { translation: require('./locales/zh.json') }
    }
});

const i18n = new VueI18Next(i18next);

new Vue({
    i18n: i18n,
    render: h => h(App),
}).$mount('#app')
```

引入并安装`@panter/vue-i18next`插件，`i18next.init()`是i18next的初始化配置，`lng`可以设置语言，`fallbackLng`可以设置当找不到`lng`语言对应的语言资源时选取一个默认的资源加载，然后创建`i18n`对象并注入全局。

我们在`locales`文件夹下新建两个语言资源文件:`en.json`和`zh.json`。

``` json
// en.json
{
    "hello": "Welcome to Your <i>Vue.js</i> App",
    "name": "blackstar",
    "age": "Age: {{num}}"
}
```

``` json
// zh.json
{
    "hello": "欢迎来到你的<i>Vue.js</i>项目",
    "name": "黑星",
    "age": "年龄: {{num}}"
}
```

在`Age: {% raw %}{{% endraw %}{num}}`这种写法中，`num`为传入的变量。

把这两个资源文件配置到`resources`后我们的准备工作就都完成了。

#### 使用i18next实现国际化

``` html
<h1 v-html="$t('hello')"></h1>
<h2>{{ $t(name) }}</h2>
<p>{{ $t('age', {num: 25}) }}</p>
```

分别将语言设置为中文和英文我们可以得到对应的语言页面:

![i18](https://s2.ax1x.com/2019/02/18/kc6fcq.png)

`$t()`为全局方法用以转换语言键对应的内容，上面展示了i18next的普通输出、输出html标签以及传参的基本用法，不仅如此我们还可以在js中使用i18next:

``` javascript
export default {
    name: 'HelloWorld',
    props: {
        msg: String
    },
   	data() {
        return{
            name: ''
        }
    },
    mounted(){
        this.name = this.$t('name')
    }
}
```

#### 切换语言

既然做国际化，那么切换语言的功能必不可少。我们在页面添加一个按钮，再写一个方法:

``` javascript
changeLng() {
    // 获取当前语言
    let curLng = this.$i18n.i18next.language
    // 切换语言
    this.$i18n.i18next.changeLanguage(curLng.indexOf('en')>-1?'zh':'en');
}
```

`i18next.language`获取当前语言，`i18next.changeLanguage()`方法用来切换语言。使用效果如下:

![change-lng](https://s2.ax1x.com/2019/02/19/kghm01.gif)

#### i18next插件

i18next还提供了各式的插件用于满足开发者和用户的需求，这里介绍两个比较常用的。

为了避免一个资源文件后期过于庞大不便维护，我们可以将其按照功能模块拆分成多个文件，那么就不能在`resources`里只引入一个文件，所以我们需要`i18next-xhr-backend`。同时我们希望网页可以显示浏览器设置的语言，i18next也提供了一个名为`i18next-browser-languagedetector`的插件。安装:

``` bash
yarn add i18next-xhr-backend
yarn add i18next-browser-languagedetector
```

配置:

``` javascript
import Vue from 'vue'
import App from './App.vue'
import i18next from 'i18next';
import VueI18Next from '@panter/vue-i18next';
import XHR from 'i18next-xhr-backend';
import LngDetector from 'i18next-browser-languagedetector';

Vue.use(VueI18Next);

Vue.config.productionTip = false

i18next.use(XHR).use(LngDetector).init({
    // lng: 'zh', // 设定语言
    fallbackLng: 'en', // 默认语言包
    ns: ['user', 'main'],
    defaultNS: 'user',
    backend: {
        loadPath: 'locales/{{lng}}/{{ns}}.json'
    },
    detection: {
        // order and from where user language should be detected
        order: ['querystring', 'navigator'],
        // cache user language on
        caches: ['localStorage', 'cookie']
    }
});

const i18n = new VueI18Next(i18next);

new Vue({
    i18n: i18n,
    render: h => h(App),
}).$mount('#app')
```

![backend](https://s2.ax1x.com/2019/02/19/kgI8cF.png)

`ns`表示加载的资源文件名数组集合，要注意语言资源的存放路径要在域名下可直接访问，`loadPath`为资源路径。语言资源变成多文件后，使用时也要相应调整:

``` html
<h2>{{ $t('user:name') }}</h2>
```

`defaultNS`表示如果不加`user:`这样的文件名则默认去指定的文件寻找键值。

`detection`下为`i18next-browser-languagedetector`的配置，`order`设置从何处读取语言，除了上面列出的url参数和浏览器设置，还有`cookie`、`localStorage`等；`caches`表示以何种方式缓存语言设置。

### Angular下使用i18next

其实我工作中使用i18next最多的框架就是Angular，但是该项目是基于Angular1.5的，实在是太**古老**了，关于Angular的i18next插件有三个:

![support_i18](https://s2.ax1x.com/2019/02/19/kgHPbQ.png)

三种插件的配置差别有点大，具体可以到他们的github上查看，在这里只贴出`ng-i18next`的配置:

``` javascript
window.i18next
    .use(window.i18nextXHRBackend)
    .use(window.i18nextBrowserLanguageDetector);
window.i18next.init({
    debug: true,
    ns: ['i', 'member'],
    detection: {
        order: ['localStorage', 'querystring', 'navigator']
    },
    escapeInterpolation: false,
    defaultNS: 'i',      
    //   lng: 'en', // If not given, i18n will detect the browser language.
    fallbackLng: 'en', // Default is dev
    backend: {
        loadPath: 'locales/{{lng}}/{{ns}}.json'
    },
    useCookie: false,
    useLocalStorage: false
}, function (err, t) {

});
```

使用上`ng2-i18next`类似于Vue，通过`i18n.t('hello')`调用，`ng-i18next`和`angular-i18next`则基本相同，几种常用的用法如下:

``` html
<p>{{'member:common.currency' | i18next}}</p>
<p>{{'member:common.currency' | i18next: {num: value} }}</p>  // 注意变量的大括号和i18的双括号之间有空格，否则会报错
<p ng-i18next="[html]member:common.currency"></p>
<p ng-i18next="[html]({num: value})member:common.currency"></p> // 可解析html标签并传入参数
```

在js中可使用`i18next.t('member:common.currency')`，这是个全局方法。

### 结语

如果你只是要对项目进行基础的国际化那么本篇文章的内容应该足够你使用了，若还需更深入的拓展也可以到官网上查询API及其余插件的用法。