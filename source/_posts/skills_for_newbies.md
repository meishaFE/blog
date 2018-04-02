---
title: 前端新人入职需快速掌握的几个技能
date: 2018-03-19 20:00:00
tags: js、html、css
---

## 前言

本来想写一篇《前端新人常会遇到的十个坑》。

后来想想，其实很多时候新人遇到的坑各种各样，但经过总结归纳后基本上都是相类似的问题，而且其实大多数新人所遇到坑都不能称之为真正意义上的坑（例如使用了某组件库中确实存在有bug的组件），而是由于对前端技术的掌握程度不够而产生的误区。

俗话说的好：授人以鱼不如授人以渔，后来便将标题修改为《前端新人入职需快速掌握的几个技能》。

<!-- more -->
<br>

## 一、ES6语法 （ES7、8已出）

可能有人问为什么要用ES6语法？ 
回答那当然是因为用着会更加方便，更加简洁，ES6而且现在很多转换器已经可以把ES6所有的特性以及ES7的部分特性转换为ES5，其中Babel就是一个非常好的转换器。

**对于入门ES6可以从以下十个最基础的特性入手：**

> * 1、Default Parameters（默认参数）
> * 2、Template Literals （模板文本）
> * 3、Multi-line Strings （多行字符串）
> * 4、Destructuring Assignment （解构赋值）
> * 5、Enhanced Object Literals （增强对象字面量）
> * 6、Arrow Functions （箭头函数）
> * 7、Promises
> * 8、Block-Scoped Constructs Let and Const（块作用域构造Let and Const）
> * 9、Classes（类）
> * 10、Modules（模块）


<br>
## 二、前端框架的使用

最常用的基于mvvm思想设计的前端框架包括vue、react、angular。

想要用好一个前端框架，应该也对其配套工具有一定程度的掌握
本文将以Vue作为例子进行描述：

使用Vue之前，当然要先了解它的生命周期。而了解Vue生命周期的关键在于认识什么是Vue实例，每进行一次new Vue()都会创建一个Vue实例。

**Vue的生命周期包括(Vue生命周期从开始到结束的排序依次为)：**
> * 1、beforeCreated（在实例初始化操作之后，实例创建完成之前调用）
> * 2、created（在实例创建完成后立即调用。ps：此时虚拟$el属性不可见）
> * 3、beforeMounted（实例创建完成之后，实例挂载到dom之前）
> * 4、mounted（真实的el被虚拟$el替换，并挂载到dom。值得注意的是，在父组件挂载成功的瞬间，其子组件不一定也都完成挂载）
>   * 4.1、beforeUpdate （data数据更新的时调用，在虚拟dom重新渲染完成前调用）
>   * 4.2、update（data数据更新导致的虚拟dom重新渲染完成后调用）
> * 5、beforeDestroy（实例销毁之前调用）
> * 6、destroyed（实例销毁后调用）

**几个钩子：**
> * 1、activated（使用了keep-alive的组件在激活时调用。）
> * 2、deactivated（使用了keep-alive的组件在停用时调用。）


**Vue全家桶，包括：**
> * 1、vue-cli：vue-cli是vue.js提供的一个官方命令行工具，可用于快速搭建一个带热重载、保存时静态检查大型单页应用
> * 2、vue-router：vue-router用于创建单页应用，只需要将组件映射到相应的routes即可。
> * 3、vuex：vuex是一个状态管理模式，用于集中存储和管理应用中的所有组件的状态
> * 4、vue-resource：vue-resource可以用于发起请求并处理。此外，它还提供了拦截器（interceptor）的功能。

(Ps: 还有个用于调试vue的插件：vue Devtools)
<br>

## 三、（基于Vue的）组件库：
**PC端：**
Element、iView、VueStrap、Vue Admin、Keen UI、Vue MDL、BootstrapVue、Vue-Blu、VueKit

**移动端：**
Vux、Mint UI、Vonic、Vum、Vue-Carbon、YDUI、Vuwe、WE-VUE

一般来说，强大的组件库已经能够基本满足我们的要求。但也不是说组件库就是万能的，有时候组件库也会出现bug。这种情况下可以去github的issue上查找其他使用者有没有出现过类似的情况。其中有很多情况都可以通过升级组件库版本解决，但有时候则需要通过自己对组件进行问题排查，再另加逻辑或样式来修复bug。
<br>

## 四、CSS BEM书写规范 http://getbem.com/naming/

BEM（Block, Element, Modifier）是一种基于组件化开发的CSS命名规范，通过用模块，元素以及修饰符来定位到具体的一个部分。
如：
> * 1、.block代表了更高级别的抽象或组件（块）。
> * 2、.block__element 代表.block的后代，用于形成一个完整的.block的整体（元素）。
> * 3、.block--modifier代表.block的不同状态或不同版本（修饰）。

**Block**
```html
<div class="block">...</div>  <!--任何一个具有类名的dom节点都可以作为一个块-->
```
**Element**
```html
<div class="block">
  ...
  <span class="block__elem"></span> <!--任何一个块内的dom节点都可以作为一个元素-->
</div>
```

**Modifier**
```html
<!--Modifier的正确使用-->
<div class="block block--mod">...</div> <!--Modifier是一个加在块或者元素dom节点上的一个额外的类名-->
	<div class="block block--size-big
    block--shadow-yes">...</div>

<!--Modifier的错误使用-->
<div class="block--mod">...</div> <!--Modifier不应该这么单独使用在一个dom节点上-->
```


BEM规范使得所有的命名之间的联系变的更加紧密，可以仅凭名字从而知道该样式为哪个块（组件）所使用
<br>

## 五、公共逻辑部分的模块化

在编写js的时候要有将公用部分模块化的意识。

在ES6之前，前端一般会使用RequireJS 或者 seaJS来实现模块化。在出了ES6之后，我们可以直接使用ES6语法的import和export进行项目公共部分的模块化。

**合理的模块化是为了更好的管理我们的项目，对于公用的部分，如果在多个地方重复加同样的代码，会带来很多问题。**
一、会造成项目代码冗余；
二、当使用到的公共逻辑部分发生改动，那么很多地方可能都需要做相应的联动修改，浪费了大部分开发的时间。

**一般项目中经常会用到的公用工具部分包括（但不限于）：**
> * 1、与后端交互的api接口 （api.js）
> * 2、公用ajax方法 （common.js）
> * 3、本地封装好的ajax方法 （fetch.js）
> * 4、过滤器 （filter.js）
> * 5、正则匹配方法 （reg.js）
> * 6、类型判断 （type.js）

![](./folder_structure.png '公用方法的提取')


<br>

## 六、打包工具的使用（webpack）

**1、什么是webpack**
webpack正如它的名字--“打包网页”，是一个用来打包网页项目的工具。目前主流的打包工具有webpack，RequireJS、browserify等等。

**2、为什么选择webpack**
webpack有两个优点（因人而异）：require anything和code splitting

**require anything：**
在webpack中，不管是js，css，less，scss，jade，png，各种各样类型的文件都可以被当作模块。对应不同的文件类型的资源，webpack都有相应的模块加载器loader。

```javascript
module: {
	//加载器配置
	loaders: [
		//.css 文件使用 style-loader 和 css-loader 来处理
		{ test: /\.css$/, loader: 'style-loader!css-loader' },
		//.js 文件使用 jsx-loader 来编译处理
		{ test: /\.js$/, loader: 'jsx-loader?harmony' },
		//.scss 文件使用 style-loader、css-loader 和 sass-loader 来编译处理
		{ test: /\.scss$/, loader: 'style!css!sass?sourceMap'},
		//图片文件使用 url-loader 来处理，小于8kb的直接转为base64
		{ test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'}
	]
}
```

**code splitting：**
按照官方的说法，code splitting有两个特点：
1、避免代码冗余
2、动态加载（按需加载）
通过code splitting,可以避免对于一个大型的网站一次性加载所有代码所带来的加载缓慢的问题。

<br>

## 七、遇到问题如何解决

首先第一点当然是： 不要放弃治疗！
如果你想从事前端领域，但如果没有一颗大心脏，禁受不住各种不断地掉坑和爬坑，那我建议你还是去当产品经理吧（just joking ! ^_^）

一般来说，新手遇到问题的时候，最简单粗暴的办法就是 阅读相关的技术文档！！！ 我们经常会在还没有足够了解一个技术的时候，就着手去写代码，遇到问题的时候会特别受挫。 所以最关键的一点就是阅读技术文档（虽然有的技术文档写的是真的一般）。
对于前端而言，还有相当多的途径可以解决你遇到的问题。例如GITHUB，STACKOVERFLOW，SEGMENTFAULT，还有各种各样的博客、社区等等。这上面的途径基本上80%，90%囊括了你将会遇到的问题。

**如果这样还不能解决你的问题的话，那么，欢迎投简历到hr@meishakeji.com，我们将一一为你解答。**