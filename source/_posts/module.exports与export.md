---
title: module.exports与export
categories:
  - - 模块化
tags:
  - JavaScript模块化
  - ES6
  - CommonJs
date: 2019-08-04 14:00:00
---
## 前言

在平时的项目开发中，经常会见到类似 `module.exports`、`exports`、 `export`、`exports default`、`require`、`import` 这样的字眼，那到底这几个看上去这么相似的关键字分别是在什么时候用到的呢？要搞清楚这几个关键字，就得对 `CommonJs` 和 `ES6` 的模块化规范有基础的认识。

<!-- more -->

## 1. `CommonJs`模块化规范


> CommonJS是一个项目，其目标是为JavaScript在网页浏览器之外创建模块约定。创建这个项目的主要原因是当时缺乏普遍可接受形式的JavaScript脚本模块单元，模块在与运行JavaScript脚本的常规网页浏览器所提供的不同的环境下可以重复使用。
> [from 维基百科](https://zh.wikipedia.org/wiki/CommonJS)

CommonJs的模块化规范的应用最常见的是在Node中，基于CommonJs规范来管理每个模块（文件），规范主要有以下几点：

> * 每个文件是一个模块，有单独的作用域，定义的变量是私有的，不会造成全局污染。
> * 在每个模块内部具有一个对象，即module，表示当前模块，并提供了一个exports的接口输出内容。
> * 使用 `require` 加载对应模块输出的内容。（require可以加载`.js`、`.json`、`.node`后缀的文件）

例子：  
```js
// test.js
let a = 123;
let b = 321;
module.exports.a = a;
module.exports.b = b;

// index.js
let a = require('./test.js');
console.log(a); // a { a: 123, b: 321 }
```



## 2. `ES6`模块化规范

对于ES6的模块化，则有：

> * 与CommonJs相比，ES6的模块化方式是通过 `import` 来导入模块， `export` 来提供输出接口（使用export导出指定的变量）。
> * ES6无法import除了JS以外的文件（但是可以结合Webpack来实现）

例子1：

```js
// test.js

let a = 123;
let b = 321;

export { a, b }

// index.js
import { a, b } from './test.js';
```
或者

例子2：
```js
// test.js

let a = 123;
let b = 321;

export default {
  a,
  b
}

// index.js
import obj from './test.js';
console.log(obj); // { a: 123, b: 321 }
```

其中：例子1和例子2的`export`可以结合使用，主要看实际需要导入哪些变量来决定如何import（但在每个单独的文件中对一个文件只能执行一次import）

例子3:

```js
// test.js
let a = 123;
export { a as b}; // 将b作为a导出

// index.js
import { b } from './test.js';
console.log(b); // 123
```

## 3. 编译时加载（ES6） 与 运行时加载（CommonJs）

编译时加载 是相对 运行时加载 而言的（ps: 编译阶段早于运行阶段）

> * 编译时加载，可以理解为在代码编译的阶段（变量的值还未确定的时候），就已经将变量引入到当前环境中
> * 运行时加载，顾名思义，即在代码运行的时候才进行对变量的引入

编译时加载 与 运行时加载 最大的区别在于：

运行时加载是同步加载所依赖的模块中的变量，模块内的异步代码执行不会影响输出的变量的值；而编译时加载只是输出的值的引用。

例子（伪代码，方便理解）：

```js
// 编译时加载
// test.js
let a = 123;
export a;
setTimeout(() => {
  window.a = 321;
}, 1000)

// index.js
import { a } from './test.js';
console.log(a); // 123
setTimeout(() => {
  console.log(window.a); // 321
}, 2000)

```

```js
// 运行时加载
// test.js
let a = 123;
module.exports = a;
setTimeout(() => {
  window.a = 321;
})

// index.js
let a = require('./test.js');
console.log(a); // 123
setTimeout(() => {
  console.log(window.a); // 123
})
```

`最后，如有理解错误的地方和不足的地方，欢迎提出和指正。`
