---
title: koa 洋葱模型简析
date: 2019-03-04 17:50:36
tags:
- node.js
- koa
---

## 前言
`koa` 是现在主流的的 node.js web framework，其主要的特点就是独特的中间件流程控制，是一个典型的洋葱模型。`koa` 和 `koa2` 中间件的思路是一样的，不过实现方式有所区别，`koa2` 在 node7.6 之后可以直接用 `async/await` 来替代 `generator` 使用中间件。那么洋葱模型是什么呢？

<!-- more -->

## 洋葱模型
洋葱模型的原理是：把后一个函数当做参数传递给前一个函数，从而实现把多个函数串起来执行的效果。

![koa-onion](https://cdn.meishakeji.com/blog/koa-onion-model.png)

举个例子
```js
const Koa = require('koa');

const app = new Koa();
const PORT = 3000;

// #1
app.use(async (ctx, next)=>{
    console.log(1)
    await next();
    console.log(5)
});
// #2
app.use(async (ctx, next) => {
    console.log(2)
    await next();
    console.log(4)
})

app.use(async (ctx, next) => {
    console.log(3)
})

app.listen(PORT);
console.log(`http://localhost:${PORT}`);
```

这里的输出结果应该是

```
1
2
3
4
5
```

## koa2 源码分析

查看一下 koa2 源码可知 `app.listen` 使用了`this.callback()` 来生成node的 `httpServer` 的回调函数

```js
listen(...args) {
    debug('listen');
    const server = http.createServer(this.callback());
    return server.listen(...args);
  }
```

那么看一下 `callback` 做了什么

```js
callback() {
    const fn = compose(this.middleware);

    if (!this.listenerCount('error')) this.on('error', this.onerror);

    const handleRequest = (req, res) => {
      const ctx = this.createContext(req, res);
      return this.handleRequest(ctx, fn);
    };

    return handleRequest;
  }
```

这里用 `koa-compose` 处理了一下 `this.middleware`，创建了 `ctx` 并赋值为`createContext` 的返回值，最后返回了 `handleRequest`。

this.middleware 是中间件的数组集合

```js
use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
    if (isGeneratorFunction(fn)) {
      deprecate('Support for generators will be removed in v3. ' +
                'See the documentation for examples of how to convert old middleware ' +
                'https://github.com/koajs/koa/blob/master/docs/migration.md');
      fn = convert(fn);
    }
    debug('use %s', fn._name || fn.name || '-');
    this.middleware.push(fn);
    return this;
}

// 其实上面的代码主要做了一件事

use(fn) {
  // ...
  this.middleware.push(fn);
  return this;
}
```

`koa-compose` 主要对 `middleware` 做了封装，将 `context` 一路传给中间件，同时将 `middleware` 中的下一个中间件 `fn` 作为未来 `next` 的返回值

```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */

  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

这部分代码也就是洋葱模型的核心。
