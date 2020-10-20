# 1 nuxt.js概述

nuxt.js简单来说是Vue的通用框架，最常用的就是用来作SSR（服务器端渲染），Nuxt.js这个框架，用Vue开发多页应用，并在服务端完成渲染，可以直接用命令把我们制作的vue项目生成为静态html。

## 1.1 什么是SSR？

在认识`SSR`之前，首先对`CSR`与`SSR`之间做个对比。

首先看一下传统的web开发，传统的web开发是，客户端向服务端发送请求，服务端查询数据库，拼接`HTML`字符串（模板），通过一系列的数据处理之后，把整理好的`HTML`返回给客户端,浏览器相当于打开了一个页面。这种比如我们经常听说过的`jsp`,`PHP`,`aspx`也就是传统的`MVC`的开发。

`SPA`应用，到了`Vue`、`React`，单页面应用优秀的用户体验，逐渐成为了主流，页面整体式`javaScript`渲染出来的，称之为客户端渲染`CSR`。`SPA`渲染过程。由客户端访问`URL`发送请求到服务端，返回`HTML`结构（但是`SPA`的返回的`HTML`结构是非常的小的，只有一个基本的结构，如第一段代码所示）。客户端接收到返回结果之后，在客户端开始渲染`HTML`，渲染时执行对应`javaScript`，最后渲染`template`，渲染完成之后，再次向服务端发送数据请求，注意这里时数据请求，服务端返回`json`格式数据。客户端接收数据，然后完成最终渲染。

`SPA`虽然给服务器减轻了压力，但是也是有缺点的：

1. 首屏渲染时间比较长：必须等待`JavaScript`加载完毕，并且执行完毕，才能渲染出首屏。
2. `SEO`不友好：爬虫只能拿到一个`div`元素，认为页面是空的，不利于`SEO`。

为了解决如上两个问题，出现了`SSR`解决方案，后端渲染出首屏的`DOM`结构返回，前端拿到内容带上首屏，后续的页面操作，再用单页面路由和渲染，称之为服务端渲染(`SSR`)。

`SSR`渲染流程是这样的，客户端发送`URL`请求到服务端，服务端读取对应的`url`的模板信息，在服务端做出`html`和`数据`的渲染，渲染完成之后返回`html`结构，客户端这时拿到的之后首屏页面的`html`结构。所以用户在浏览首屏的时候速度会很快，因为客户端不需要再次发送`ajax`请求。并不是做了`SSR`我们的页面就不属于`SPA`应用了，它仍然是一个独立的`spa`应用。

`SSR`是处于`CSR`与`SPA`应用之间的一个折中的方案，在渲染首屏的时候在服务端做出了渲染，注意仅仅是首屏，其他页面还是需要在客户端渲染的，在`服务端`接收到请求之后并且渲染出首屏页面，会携带着剩余的路由信息预留给`客户端`去渲染其他路由的页面。

Nuxt.js是特点（优点）：

- 基于`Vue`
- 自动代码分层
- 服务端渲染
- 强大的路由功能，支持异步数据
- 静态文件服务
- `EcmaScript6`和`EcmaScript7`的语法支持
- 打包和压缩`JavaScript`和`Css`
- `HTML`头部标签管理
- 本地开发支持热加载
- 集成`ESLint`
- 支持各种样式预编译器`SASS`、`LESS`等等
- 支持`HTTP/2`推送

# 2 Nuxt应用程序安装

```
npm i create-nuxt-app -g
create-nuxt-app my-nuxt-demo
cd my-nuxt-demo
npm run dev
```

安装向导：

```
Project name                                //  项目名称
Project description                         //  项目描述
Use a custom server framework               //  选择服务器框架
Choose features to install                  //  选择安装的特性
Use a custom UI framework                   //  选择UI框架
Use a custom test framework                 //  测试框架
Choose rendering mode                       //  渲染模式
Universal                                   //  渲染所有连接页面
Single Page App                             //  只渲染当前页面
```

## 3.Nuxt 渲染流程

一个完整的服务器请求到渲染的流程



![img](https://user-gold-cdn.xitu.io/2019/4/30/16a6db58678e3bbe?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



通过上面的流程图可以看出，当一个客户端请求进入的时候，服务端有通过`nuxtServerInit`这个命令执行在`Store`的`action`中，在这里接收到客户端请求的时候，可以将一些客户端信息存储到`Store`中，也就是说可以把在服务端存储的一些客户端的一些登录信息存储到`Store`中。之后使用了`中间件`机制，中间件其实就是一个函数，会在每个路由执行之前去执行，在这里可以做很多事情，或者说可以理解为是路由器的拦截器的作用。然后再`validate`执行的时候对客户端携带的参数进行校验，在`asyncData`与`fetch`进入正式的渲染周期，`asyncData`向服务端获取数据，把请求到的数据合并到`Vue`中的`data`中。

# 3 Nuxt目录结构

## # 目录结构介绍

```
└─my-nuxt-demo
  ├─.nuxt               // Nuxt自动生成，临时的用于编辑的文件，build
  ├─assets              // 用于组织未编译的静态资源如LESS、SASS或JavaScript,对于不需要通过 Webpack 处理的静态资源文件，可以放置在 static 目录中
  ├─components          // 用于自己编写的Vue组件，比如日历组件、分页组件
  ├─layouts             // 布局目录，用于组织应用的布局组件，不可更改⭐
  ├─middleware          // 用于存放中间件
  ├─node_modules
  ├─pages               // 用于组织应用的路由及视图,Nuxt.js根据该目录结构自动生成对应的路由配置，文件名不可更改⭐
  ├─plugins             // 用于组织那些需要在 根vue.js应用 实例化之前需要运行的 Javascript 插件。
  ├─static              // 用于存放应用的静态文件，此类文件不会被 Nuxt.js 调用 Webpack 进行构建编译处理。 服务器启动的时候，该目录下的文件会映射至应用的根路径 / 下。文件夹名不可更改。⭐
  └─store               // 用于组织应用的Vuex 状态管理。文件夹名不可更改。⭐
  ├─.editorconfig       // 开发工具格式配置
  ├─.eslintrc.js        // ESLint的配置文件，用于检查代码格式
  ├─.gitignore          // 配置git忽略文件
  ├─nuxt.config.js      // 用于组织Nuxt.js 应用的个性化配置，以便覆盖默认配置。文件名不可更改。⭐
  ├─package-lock.json   // npm自动生成，用于帮助package的统一设置的，yarn也有相同的操作
  ├─package.json        // npm 包管理配置文件
  ├─README.md
```

## # 配置文件

```js
const pkg = require('./package')
module.exports = {
  mode: 'universal',    //  当前渲染使用模式
  head: {       //  页面head配置信息
    title: pkg.name,        //  title
    meta: [         //  meat
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: 'description', name: 'description', content: pkg.description }
    ],
    link: [     //  favicon，若引用css不会进行打包处理
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
    ]
  },
  loading: { color: '#fff' },   //  页面进度条
  css: [    //  全局css（会进行webpack打包处理）
    'element-ui/lib/theme-chalk/index.css'  
  ],
  plugins: [        //  插件
    '@/plugins/element-ui'
  ],
  modules: [        //  模块
    '@nuxtjs/axios',
  ],
  axios: {},
  build: {      //  打包
    transpile: [/^element-ui/],
    extend(config, ctx) {       //  webpack自定义配置
    }
  }
}

```

## # Nuxt运行命令

```js
{
  "scripts": {
    //  开发环境
    "dev": "cross-env NODE_ENV=development nodemon server/index.js --watch server",
    //  打包
    "build": "nuxt build",
    //  在服务端运行
    "start": "cross-env NODE_ENV=production node server/index.js",
    //  生成静态页面
    "generate": "nuxt generate"
  }
}
```

# 4 静态资源和打包

## 4.1 静态资源

```js
（1）直接引入图片
在网上任意下载一个图片，放到项目中的static文件夹下面，然后可以使用下面的引入方法进行引用
<div><img src="~static/logo.png" /></div>
```

“~”就相当于定位到了项目根目录，这时候图片路径就不会出现错误，就算打包也是正常的。

```js
（2）CSS引入图片
如果在CSS中引入图片，方法和html中直接引入是一样的，也是用“~”符号引入。
复制代码
<style>
  .diss{
    width: 300px;
    height: 100px;
    background-image: url('~static/logo.png')
  }
</style>
```

这时候在npm run dev 下是完全正常的。

## 2.打包

用Nuxt.js制作完成后，你可以打包成静态文件并放在服务器上，进行运行。

在终端中输入：

```js
npm run generate
```

然后在dist文件夹下输入live-server就可以了。

tips：有更新之后在服务器上面要记得node install一下，然后再打包就可以运行，具体流程是：



```js
ssh 120.78.139.247 -p 56000
找到你放的静态文件包
netstat -tunlp|grep 8089 // 查看当前node服务的进程
kill掉当前服务的进程
运行 nohup npm start &然后回车输出exit //不主动exit的话关掉shell时进程会中断
```



总结：Nuxt.js框架非常简单，因为大部分的事情他都为我们做好了，我们只要安装它的规则来编写代码。