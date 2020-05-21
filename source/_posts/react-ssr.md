---
title: React 服务端渲染
date: 2019-12-08 20:00:00
tags:
  - react
---

虽然 react 等框架有极大的便利性，但是现在还是有服务端渲染的需求，比如 SEO、加快首屏渲染速度等。那么如何既能使用 react 又能实现服务端渲染呢？目前也有相关的框架，比如 next.js(对应 react 的应用)和 nuxt.js(对应 vue 的应用)。

下面的主题就是去从 0 开始搭建一套 React 技术栈下的 SSR 框架，深入了解一下 SSR 框架的原理。

<!-- more -->

本项目的 demo 地址 [https://github.com/wawjqyh/demo-react-ssr](https://github.com/wawjqyh/demo-react-ssr)

# 1 什么是服务端渲染

页面的渲染模式有两种，分别是客户端渲染(CSR)和服务端渲染(SSR)。

## 1.1 服务端渲染(SSR)

> 服务端渲染概念： 是指，浏览器向服务器发出请求页面，服务端将准备好的模板和数据组装成完整的 HTML 返回给浏览器展示

在还没有前端这个职业（前端还是切图仔）的时候，通常是后端包揽了整个网站的开发。使用 ASP、Java、PHP 做后端渲染。

## 1.2 客户端渲染(CSR)

什么是客户端渲染呢？随着 ajax 技术的普及以及前端框架的崛起(jq、Angular、React、Vue) 框架的崛起，开始转向了前端渲染,使用 JS 来渲染页面大部分内容达到局部刷新。就是后台只提供接口，前端只负责页面部分，页面内的数据通过 ajax 调用后端的接口然后由前端渲染出来。像现在主流的三大框架(react vue angular)都是客户端渲染的。

### 1.2.1 客户端渲染的缺点

第一个是首屏加载时间(TTFP)比较慢。客户端渲染的流程：

`浏览器下载html文档 => 下载js => 运行js代码 => 渲染页面`

而且客户端渲染首屏加载的资源都比较多，如果没有做按需加载的话几乎把除多媒体外的整个网站都加载了。当然加载了首屏后跳转其他页面就很快了。

第二个就是不支持 SEO，这也是重要的一个点，如果一个网站要做 seo 的话不能用客户端渲染的方式（其实也可以，可以用预渲染的方式解决）。

# 2 SSR 与 CSR 实例

下面通过实例更直观的理解服务端渲染与客户端渲染。

## 2.1 客户端渲染

通过 `create-react-app` 创建一个 react 项目（或者其他工具，这里用的是 umi），直接在开发环境启动，下面是项目的首页：

```javascript
class Index extends React.Component {
  render() {
    return (
      <div>
        <h1>客户端渲染</h1>
        <p>hello world</p>
      </div>
    );
  }
}
```

在浏览器中查看，页面已经展示出了内容：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/2_aboutWidthCode_20191018001348.png)

再在浏览器中查看源代码，发现在里面并没有组件中的内容：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/2_aboutWidthCode_20191018001656.png)

这就是客户端渲染，浏览器最开始拿到的只是一个 html 的框架，页面的内容是通过 js 生成，而 js 是运行在浏览器上的，所以是客户端渲染。像爬虫爬到的也就是上图所看到的内容，不管页面里面有什么内容，爬虫都是拿不到的，所以客户端渲染不利于 SEO。

## 2.2 服务端渲染

服务端渲染我们得先有个服务器，这里使用 koa 创建一个（或者用 express），如果没接触过可以跟着[文档](https://koajs.com/)创建一个，比较简单。

### 2.2.1 测试

先来做个测试，给首页配个路由，返回一个字符串：

```javascript
router.get("/", (ctx, next) => {
  ctx.body = "hello";
});
```

浏览器中的效果，展示的是返回的内容：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/2_aboutWidthCode_20191018134455.png)

在浏览器中查看源代码，可以看到浏览器拿到的就是服务端返回的内容：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/2_aboutWidthCode_20191018134627.png)

### 2.2.2 返回 html

我们将返回的内容换成一串 html 字符串。

```javascript
router.get("/", (ctx, next) => {
  ctx.body = `
    <html>
      <head>
        <title>react srr</title>
      </head>
      <body>
        <h1>服务端渲染</h1>
        <p>hello</p>
      </body>
    </html>
  `;
});
```

浏览器中的效果：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/2_aboutWidthCode_20191018135430.png)

查看源代码：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/2_aboutWidthCode_20191018135540.png)

这里就实现了一个最简单的服务端渲染，可以看到服务端渲染就是浏览器能直接拿到页面的内容。

# 3 简单的 react 服务端渲染

react 服务端渲染，顾名思义就是让 react 在服务端运行，将渲染完的内容返回给浏览器，浏览器可以直接显示内容。可以理解为 react 的代码成为了后端代码，类似与模板引擎。下面通过一个很简单的例子讲解一下。

## 3.1 客户端渲染组件

首先写一个 react 组件，组件在客户端中是这么渲染的：

```javascript
import React, { Component } from "react";
import ReactDOM from "react-dom";

class Index extends Component {
  render() {
    return (
      <div>
        <h1>react ssr</h1>
        <p>react 客户端渲染</p>
      </div>
    );
  }
}

// 客户端渲染使用 render 方法
// 这里是将虚拟 dom 转化为真实 dom 放到页面上
ReactDOM.render(<Index />, document.getElementById("root"));
```

## 3.2 服务端渲染组件

在服务端渲染也是类似的，react 提供了方法 ReactDOMServer 用于服务端渲染。下面是官网的文档：

> ReactDOMServer 对象允许你将组件渲染成静态标记。通常，它被使用在 Node 服务端上

```javascript
ReactDOMServer.renderToString(element);
```

> 将 React 元素渲染为初始 HTML。React 将返回一个 HTML 字符串。你可以使用此方法在服务端生成 HTML，并在首次请求时将标记下发，以加快页面加载速度，并允许搜索引擎爬取你的页面以达到 SEO 优化的目的。

下面是服务端渲染的写法：

```javascript
import React from "react";
import { renderToString } from "react-dom/server";
import Index from "../client/Index.jsx";

router.get("/", (ctx, next) => {
  // 这里是将虚拟 dom 转化为字符串
  const content = renderToString(<Index />);

  ctx.body = `
    <html>
      <head>
        <title>react srr</title>
      </head>
      <body>
        ${content}
      </body>
    </html>
  `;
});
```

## 3.3 使用 webpack 编译服务端代码

这时服务器是启动不了的，因为：

- 代码中使用的是 ES Module 的语法，而 node.js 遵循的是 common.js 规范
- node.js 无法直接执行 JSX 代码

所以跟客户端一样，这时服务端的代码也需要使用 webpack 编译才能运行。

webpack 的配置如下(没有其他花里胡哨的功能，只做了基础的编译 js/jsx 的配置)：

```javascript
const path = require("path");
const nodeExternals = require("webpack-node-externals");

module.exports = {
  target: "node",
  mode: "development",

  context: path.resolve(__dirname, "../"),
  entry: "./server/router.js",
  output: {
    filename: "server.js",
    path: path.resolve(__dirname, "../dist"),
    libraryTarget: "commonjs"
  },

  externals: [nodeExternals()],

  module: {
    rules: [
      {
        test: /\.(jsx|js)?$/,
        loader: "babel-loader",
        exclude: /node_modules/,
        options: {
          presets: [
            [
              "@babel/preset-env",
              { targets: { browsers: ["last 2 versions"] } }
            ],
            "@babel/preset-react"
          ]
        }
      }
    ]
  }
};
```

# 4 同构

## 4.1 同构的概念

同构的意思就是一套 react 代码，在服务端执行一次，再到客户端端执行一次。

为什么要做同构呢，先来看一个例子：

```javascript
class Index extends Component {
  render() {
    return (
      <div>
        <h1>同构</h1>
        <button
          onClick={() => {
            alert("我被点击了");
          }}
        >
          点击
        </button>
      </div>
    );
  }
}
```

上面的组件中有一个按钮，并且绑定了事件，如果使用服务端渲染点击事件不会被触发，因为 `renderToString` 只是将虚拟 dom 转成字符串返回，所以点击事件根本没有绑定。所以这里就需要使用同构，需要将 react 代码再到浏览器执行一次。

使用查看源码查看服务器返回的内容（可以看到并没有绑定事件）：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/4_isomorphism_20191022235440.png)

## 4.2 如何实现同构

实现同构也就是既要服务器渲染也要客户端渲染，服务端渲染已经实现了，客户端渲染就跟传统的 react 项目一样，在 html 中引入打包后的 js 文件。

不同的是渲染要的方法不一样，同构使用的是 `hydrate` 方法。`render` 方法会再次渲染页面，因为服务端渲染已经拿到了页面的内容，不需要再完整的渲染一次，只需要做一些事件的绑定。

下面摘抄自 react 官网：

```javascript
ReactDOM.hydrate(element, container[, callback])
```

> 与 render() 相同，但它用于在 ReactDOMServer 渲染的容器中对 HTML 的内容进行 hydrate 操作。React 会尝试在已有标记上绑定事件监听器。

注意点：

- 客户端和服务端的代码都需要使用 webpack 打包
- 客户端和服务端使用的是同一套代码（都是调用的 client/pages 下的组件），唯一不同的是 rander 方法不一样
- 需要在 html 中引入打包后的 js 文件

下面是项目的结构（注意这是一个最简单的演示项目，实际项目自行完善）：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/4_isomorphism_20191028005819.png)

```javascript
// src/main.jsx 代码

const appTarget = document.getElementById("root");
// 同构需要使用 hydrate
ReactDOM.hydrate(<Index />, appTarget);
```

```javascript
// server/router.js 代码
router.get("/", (ctx, next) => {
  const content = renderToString(<Home />);

  ctx.body = `
    <html>
      <head>
        <title>react srr</title>
      </head>
      <body>
        <div id="root">${content}</div>
        <script src="/client.js"></script>
      </body>
    </html>
  `;
});

// <div id="root"></div> 中间不要有空格或者换行，不然会有警告
// 在 body 中引入打包后的客户端 js
```

# 5 在 SSR 中使用路由

首先看本次用例的项目结构：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/5_router_20191028231319.png)

下面是一个配置路由的组件，客户端和服务端都是调用这个组件：

```javascript
// src/router.jsx
// 路由配置文件
class RouterIndex extends Component {
  render() {
    return (
      <React.Fragment>
        <Route exact path="/" component={Index} />
        <Route exact path="/login" component={Login} />
        <Route exact path="/hello" component={Hello} />
      </React.Fragment>
    );
  }
}
```

客户端和服务端使用路由的方式区别不大，客户端渲染使用的是 `BrowserRouter`。

如下面的代码：

```javascript
// src/main.jsx
// 客户端使用路由
const appTarget = document.getElementById("root");
ReactDOM.hydrate(
  <BrowserRouter>
    <RouterIndex />
  </BrowserRouter>,
  appTarget
);
```

而服务端渲染使用的是 `StaticRouter`，代码如下：

```javascript
// server/router.js
// 服务端使用路由
router.get("*", (ctx, next) => {
  const content = renderToString(
    <StaticRouter location={ctx.request.url} context={{}}>
      <RouterIndex />
    </StaticRouter>
  );

  ctx.body = `
    <html>
      <head>
        <title>react srr</title>
      </head>
      <body>
        <div id="root">${content}</div>
        <script src="/client.js"></script>
      </body>
    </html>
  `;
});
```

注意点：

- `router.get('*')` 服务端需要匹配所有的路由
- `StaticRouter` 需要传入服务端所匹配到的地址。`BrowserRouter` 可以直接拿到当前地址，因为运行在浏览器端

> 因为使用了同构，服务端渲染只渲染第一次请求的页面

> 页面在浏览器运行后，后面的路由跳转和页面渲染就会由客户端接管，因为单页面应用路由跳转会被拦截。

# 6 在 SSR 中使用 redux

首先抛出几个问题：

1. 服务端如何使用 redux，如何构建代码
2. 服务端如何拿到异步的数据
3. 如何做同构

## 6.1 写一个简单的 redux

下面的代码注册了一个 user model，提供一个返回 store 实例的方法。

> 注意这里不能像常规的客户端渲染那样直接返回实例，因为返回的是一个单例，确保服务端每次渲染都能拿到一个新的实例

```javascript
const userInitialState = {
  name: "redux",
  desc: "这个是服务端使用 redux 的 demo"
};

const userReducer = (state, action) => {
  if (typeof state === "undefined") state = initialState;

  switch (action.type) {
    case "user/save": {
      return { ...state, ...action.payload };
    }
    default:
      return state;
  }
};

// 服务端不能直接返回 createStore，这是一个单例
export function getStore() {
  return createStore(
    combineReducers({ user: userReducer }, { user: userInitialState })
  );
}
```

在一个组件中显示 redux 中的数据（组件中有一个异步获取数据的方法，获取数据后更新 redux）：

```javascript
class Hello extends Component {
  async componentDidMount() {
    // 异步获取数据，然后更新 redux 中的数据
    const res = await getUserInfo();
    this.props.dispatch({
      type: "user/save",
      payload: res
    });
  }

  render() {
    return (
      <div>
        <div>下面是 store 中的内容</div>
        <div>{this.props.user.name}</div>
        <div style={{ color: "#f00" }}>{this.props.user.desc}</div>
      </div>
    );
  }
}

export default connect(state => ({
  user: state.user
}))(Hello);
```

## 6.2 使用 redux

### 6.2.1 SCR 中使用

使用 Provider 组件包裹需要使用 redux 的组件。

下面是关键代码：

```javascript
ReactDOM.hydrate(
  <Provider store={getStore()}>
    <BrowserRouter>
      <RouterIndex />
    </BrowserRouter>
  </Provider>,
  appTarget
);
```

### 6.2.2 SSR 中使用

同样是使用 Provider 组件～

```javascript
router.get("*", (ctx, next) => {
  const content = renderToString(
    <Provider store={getStore()}>
      <StaticRouter location={ctx.request.url} context={{}}>
        <RouterIndex />
      </StaticRouter>
    </Provider>
  );

  ctx.body = `
    <html>
      <head>
        <title>react srr</title>
      </head>
      <body>
        <div id="root">${content}</div>
        <script src="/client.js"></script>
      </body>
    </html>
  `;
});
```

## 6.3 运行效果

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/6_redux_20191107235907.png)

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/6_redux_20191106113918.png)

上面的例子中，数据获取是在客户端完成的，服务端并未获取数据。

服务端渲染渲染出了 redux 中的数据，但是只能拿到 initialState 中的数据，组件中的异步数据无法被渲染。

组件是在 componentDidMount 钩子中去获取异步数据的，但是服务端渲染不会进入这个生命周期，所以异步数据无法被获取。

那么服务端需要获取数据的话，异步方法就应该另外处理。

## 6.4 在服务端获取异步数据

关于服务端渲染获取数据，react-router-dom 文档中提供了相关的方法。

[https://reacttraining.com/react-router/web/guides/server-rendering](https://reacttraining.com/react-router/web/guides/server-rendering)

解决方案就是，需要获取数据的组件需要提供一个 loadData 的静态方法，loadData 方法是提供给服务端调用的，这个方法负责在服务器渲染之前，把这个路由需要的数据提前加载好。

而需要调用哪些 loadData 的方法，需要根据当前用户的请求地址去匹配。比如访问的是 `/login` 路径，就去拿 Login 组件的数据。所以路由的配置需要用另一种方式写。

当然不局限于一种方法，主要目的就是让服务端能够知道需要去获取哪些数据，并且提供获取数据的方法。

### 6.4.1 loadData 方法

需要服务端获取数据的组件需要提供一个 loadData 的静态方法

```javascript
// 这个方法用于服务端加载该页面的数据
Hello.loadData = async store => {
  const res = await getUserInfo();
  store.dispatch({
    type: "user/save",
    payload: res
  });
};
```

### 6.4.2 路由的配置

将每个路由的 loadData 方法写在配置文件中，改为配置式路由：

```javascript
export default [
  {
    path: "/",
    exact: true,
    component: Index,
    key: "index"
  },
  {
    path: "/login",
    exact: true,
    component: Login,
    key: "login"
  },
  {
    path: "/hello",
    exact: true,
    component: Hello,
    loadData: Hello.loadData, // 组件获取数据的方法
    key: "hello"
  }
];
```

### 6.4.3 服务端获取数据

服务端使用一个工具 [react-router-config](https://github.com/ReactTraining/react-router/tree/master/packages/react-router-config)

这个工具能够根据当前的页面地址，在路由的配置中找到当前匹配的路由。

```javascript
import { matchRoutes } from "react-router-config";
import { getStore } from "../src/store";

export default async function(url) {
  const store = getStore();
  const matched = matchRoutes(routes, url); // 匹配当前路由
  const loadDatas = []; // 加载数据

  matched.forEach(item => {
    if (item.route.loadData) loadDatas.push(item.route.loadData(store));
  });

  // 执行所有匹配到的 loadData 方法
  if (loadDatas.length) await Promise.all(loadDatas);

  const content = renderToString(
    <Provider store={store}>
      <StaticRouter location={url} context={{}}>
        {routes.map(route => (
          <Route {...route} key={route.key} />
        ))}
      </StaticRouter>
    </Provider>
  );

  return content;
}
```

### 6.4.4 运行效果

Hello 组件中 loadData 方法调用了 getUserInfo，这个方法延迟 1.5s 返回如下数据：

```javascript
export async function getUserInfo() {
  await delay(1500);
  return {
    name: "异步数据",
    desc: "这是一段描述～这里是异步的数据"
  };
}
```

在浏览器中查看源代码，服务端渲染的页面中的数据是 loadData 中返回的数据，说明服务端获取数据成功了：

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/6_redux_20191110234546.png)

查看网络请求，可以看到当前页面的请求耗时 1.51s，服务端需要等待 loadData 执行完成，拿到数据后再渲染 react 组件，饭后返回给浏览器

![](https://raw.githubusercontent.com/meishaFE/blog/master/source/images/react-ssr/6_redux_20191110234855.png)

# 7 注水和脱水

到这里服务端渲染以及基本完成，但是仔细观察页面会发现，虽然数据已经挂载到了服务端返回的 HTML 代码中，但是浏览器的初始状态是没有数据的。

这是因为当服务端拿到 store 并获取数据后，客户端的 js 代码又执行一遍，在客户端代码执行的时候又创建了一个空的 store，两个 store 的数据不能同步。

同步两个 store 的数据的操作就叫注水和脱水。

## 7.1 注水

把服务端的 store 数据注入到 window 全局环境中，就是注水操作

```javascript
<script>
  window.context = {
    state: ${JSON.stringify(store.getState())}
  }
</script>
```

## 7.2 脱水

脱水就是把 window 上绑定的数据给到客户端的 store，可以在创建 redux 实例的时候进行。

```javascript
// 这个是脱水的操作，将服务端的 state 作为 initialState 初始化 store
export function getClientStore() {
  const defaultState = window.context ? window.context : {};
  return createStore(combineReducers({ user }), defaultState);
}
```

# 8 总结

## 8.1 服务端渲染的缺点

最大的缺点就是极大的消耗服务器的性能。客户端渲染是放在用户的浏览器上，而服务端渲染是所有用户的渲染都放在了服务器上。

而运行 react 代码很消耗性能，期间需要做大量的计算、比对等。

如果服务端渲染不是非必须，即不需要做 SEO 或者对首屏的加载速度要求不高的话，不要轻易做服务端渲染。
