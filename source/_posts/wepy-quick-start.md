---
title: 使用wepy快速上手开发小程序
date: 2018-04-17 20:00:00
tags: wepy、小程序、入门
---
    - [背景](#%E8%83%8C%E6%99%AF)
    - [Vscode代码高亮设置](#vscode%E4%BB%A3%E7%A0%81%E9%AB%98%E4%BA%AE%E8%AE%BE%E7%BD%AE)
    - [微信开发者工具配置](#%E5%BE%AE%E4%BF%A1%E5%BC%80%E5%8F%91%E8%80%85%E5%B7%A5%E5%85%B7%E9%85%8D%E7%BD%AE)
    - [启动项目及项目结构](#%E5%90%AF%E5%8A%A8%E9%A1%B9%E7%9B%AE%E5%8F%8A%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84)
    - [wepy与小程序不一样的地方](#wepy%E4%B8%8E%E5%B0%8F%E7%A8%8B%E5%BA%8F%E4%B8%8D%E4%B8%80%E6%A0%B7%E7%9A%84%E5%9C%B0%E6%96%B9)
    - [wepy与vue不一样的地方](#wepy%E4%B8%8Evue%E4%B8%8D%E4%B8%80%E6%A0%B7%E7%9A%84%E5%9C%B0%E6%96%B9)
    - [常用的小程序组件的一些tip](#%E5%B8%B8%E7%94%A8%E7%9A%84%E5%B0%8F%E7%A8%8B%E5%BA%8F%E7%BB%84%E4%BB%B6%E7%9A%84%E4%B8%80%E4%BA%9Btip)
    - [使用wepy-redux进行数据管理](#%E4%BD%BF%E7%94%A8wepy-redux%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86)
    - [小程序登录逻辑](#%E5%B0%8F%E7%A8%8B%E5%BA%8F%E7%99%BB%E5%BD%95%E9%80%BB%E8%BE%91)
## 背景
公司最近需要快速开发多个小程序，为了尽量减少大家的上手时间，我们决定使用使用wepy框架来进行小程序开发。

作为一个初次接触小程序并且不是使用原生小程序框架来开发的新手，我建议大家要熟读[小程序官方文档](https://developers.weixin.qq.com/miniprogram/dev/)，熟悉小程序的逻辑层配置，熟悉小程序的API，熟悉小程序的组件，因为小程序本身的限制，文档里面有很多tip和注意事项，官方都给出了标注和避免的方法，如果没有仔细查看文档很容易在实际操作中踩坑

[wepy官方文档](https://tencent.github.io/wepy/document.html#/)对开发者非常友好，很容易上手。这篇文章的目的，是为了记录下我作为新手在初次使用wepy开发小程序过程中遇到的一些问题，其中大部分内容都是文档的摘录，然后就是我在实际开发中遇到的一些比较容易混淆和忽视的点

## Vscode代码高亮设置
我们公司前端使用的IDE是Vscode，首先要设置代码高亮，对于`.wpy`文件，设置方法如下：
1. 在 Code 里先安装 Vue 的语法高亮插件 Vetur。

2. 把.wpy 关联的语言模式选择为 Vue。

3. 因为wepy有自己的标签和像素单位， Vetur会根据vue的规则当成错误来提示，mac下使用快捷键 `⌘ ,`，打开工作区设置，粘贴下面的规则，把Vue的template和style校验关掉
```json
{
  "vetur.validation.style": false,
  "vetur.validation.template": false
}
```
另外`.wxs`文件直接关联为`js`，如果是使用原生小程序开发，`.wxml`文件关联为`html`，`wxss`文件关联为`css`

## 微信开发者工具配置
1.使用微信开发者工具新建项目，本地开发选择dist目录

2.微信开发者工具--> 项目 --> 项目设置中，取消勾选ES6 转 ES5；上传代码时样式自动补全；代码上传时自动压缩三项

3.微信开发者工具--> 项目 --> 项目设置中，勾选不校验合法域名、web-view（业务域名）、TLS 版本以及 HTTPS 证书

4.开发过程中，可以通过自定义编译，自定义场景值快速进入到某个页面开发
<img src="https://cdn.meishakeji.com/frontend-blog/img/wepy-quick-start@01.png" width="300"/>

## 启动项目及项目结构
```
npm install wepy-cli -g
wepy init standard myproject
cd myproject
wepy build --watch
```

项目目录结构
```
├── dist                   小程序运行代码目录（该目录由WePY的build指令自动编译生成，请不要直接修改该目录下的文件）
├── node_modules           
├── src                    代码编写的目录（该目录为使用WePY后的开发目录）
|   ├── components         WePY组件目录（组件不属于完整页面，仅供完整页面或其他组件引用）
|   |   ├── com_a.wpy      可复用的WePY组件a
|   |   └── com_b.wpy      可复用的WePY组件b
|   ├── pages              WePY页面目录（属于完整页面）
|   |   ├── index.wpy      index页面（经build后，会在dist目录下的pages目录生成index.js、index.json、index.wxml和index.wxss文件）
|   |   └── other.wpy      other页面（经build后，会在dist目录下的pages目录生成other.js、other.json、other.wxml和other.wxss文件）
|   └── app.wpy            小程序配置项（全局数据、样式、声明钩子等；经build后，会在dist目录下生成app.js、app.json和app.wxss文件）
└── package.json           项目的package配置
```

## wepy与小程序不一样的地方
1. 框架在app.wpy的constructor中使用中间件`this.use('promisify');`，可以对小程序提供的API全都进行`Promise`化，比如`wx.login()`可以直接写成`wepy.login()`就返回了一个Promise：`wepy.login().then(()=>{}).catch(err=>{})`

2. 框架在app.wpy的constructor中使用中间件`this.use('requestfix');`，可以修复小程序请求最大并发数为5的问题

3. 事件绑定语法使用优化语法代替

    - 原 `bindtap="click"` 替换为 `@tap="click"`，原`catchtap="click"`替换为`@tap.stop="click"`

    - 原 `capture-bind:tap="click"` 替换为 `@tap.capture="click"`，原`capture-catch:tap="click"`替换为`@tap.capture.stop="click"`

4. 事件传参使用优化后语法代替。 
    - 原`bindtap="click" data-index={{index}}`替换为`@tap="click({{index}})"`

5. 数据绑定不用通过`setData`方法，可以直接赋值，但是在异步函数中更新数据的时，必须手动调用`$apply`方法，才会触发脏数据检查流程的运行


```js
// 在原生小程序中
this.setData({title: 'this is title'});

// wepy直接赋值即可
this.title = 'this is title';

// 但是wepy在异步函数中更新数据要手动调用$apply
setTimeout(() => {
    this.title = 'this is title';
    this.$apply();
}, 3000);
```

## wepy与vue不一样的地方

1. wepy中的`methods`属性只能声明页面`wxml`标签的`bind、catch`事件，不能声明自定义方法，这与Vue中的用法是不一致的，自定义的方法应该在`methods`对象外声明，与`methods`平级

```javascript
export default class MyComponent extends wepy.component {
    // methods对象只能声明页面`wxml`标签的`bind、catch`事件
    methods = {
        bindtap () {},
        bindinput () {},
    }
    // 普通自定义方法在methods对象外声明，与methods平级
    customFunction () {}
}
```

2. 静态组件问题：WePY中的组件都是静态组件，是以组件ID作为唯一标识的，每一个ID都对应一个组件实例，当页面引入两个相同ID的组件时，这两个组件共用同一个实例与数据，当其中一个组件数据变化时，另外一个也会一起变化，如果需要避免这个问题，则需要分配多个组件ID和实例

```html
<template>
    <view class="child1">
        <child></child>
    </view>
    <view class="child2">
        <anotherchild></anotherchild>
    </view>
</template>
```
```javascript
<script>
    import wepy from 'wepy';
    import Child from '../components/child';

    export default class Index extends wepy.component {
        components = {
            //为两个相同组件的不同实例分配不同的组件ID，从而避免数据同步变化的问题
            child: Child,
            anotherchild: Child
        };
    }
</script>
```

3. 组件循环渲染，必须使用wpy的辅助标签`<repeat>`

```html
<template>
    <!-- 注意，使用for属性，而不是使用wx:for属性 -->
    <repeat for="{{list}}" key="index" index="index" item="item">
        <!-- 插入<script>脚本部分所声明的child组件，同时传入item -->
        <child :item="item"></child>
    </repeat>
</template>
```

4. 可以使用wxs替代vue中的filters
    - wxs必须是外链文件。并且后缀为.wxs。
    - wxs引入后只能在template中使用，不能在script中使用

## 常用的小程序组件的一些tip

1. `textarea、map、canvas、video` 这四个组件是客户端的原生组件，层级最高，不能通过z-index控制层级
2. `scroll-view`中不能使用滚动无法触发无法触发 `onPullDownRefresh` 事件，`scroll-view`中也不能使用1中的四个客户端原生组件
3. 除了文本节点`<text>`以外的其他节点都无法长按选中
4. 如果需要发送[模板消息](https://developers.weixin.qq.com/miniprogram/dev/api/notice.html)，需要在form表单中设置`report-submit`属性为true，收集`formId`
5. input组件是native组件，字体是系统字体，无法设置 `font-family`

## 使用wepy-redux进行数据管理

wepy支持使用`wepy-redux`来进行数据状态管理，在脚手架init的时候会询问用户是否使用redux，同意即可。`wepy-redux`主要用`connect(states, actions)`来连接组件和store，具体用法可以参考[wepy-redux文档](https://www.npmjs.com/package/wepy-redux)和脚手架init出来的demo；

wepy使用redux-actions来创建action，让action书写起来更加友好，具体使用方法也可以参考[wepy-actions文档](https://redux-actions.js.org/docs/api/)和脚手架init出来的demo


## 小程序登录逻辑

小程序的登录逻辑是比较通用的但是流程比较复杂，这里贴一下我们项目中的登录逻辑
<img src="https://cdn.meishakeji.com/frontend-blog/img/wepy-quick-start@02.png" width="600"/>

