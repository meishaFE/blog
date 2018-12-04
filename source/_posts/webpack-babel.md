---
title: webpack中babel配置
date: 2018-12-01 16:03:43
tags:
  - Webpack
---

<!-- more -->

## 1 前言

javascript 在不断的发展，各种新的标准和提案层出不穷，但是由于浏览器的多样性，导致可能几年之内都无法广泛普及，babel 可以让你提前使用这些语言特性，他是一种用途很多的 javascript 编译器，他把最新版的 javascript 编译成当下可以执行的版本，简言之，利用 babel 就可以让我们在当前的项目中随意的使用这些新最新的 es6，甚至是未正式发布的新特性(stage 0-3)

前端的同学对于 webpack 应该都比较熟悉，至少都接触过。我们这次分享的，就是如何在 webpack 中配置 babel，让它可以编译 es6。

## 2 babel 简介

Babel · 用于编写下一代 JavaScript 的编译器（来自官网）

### 2.1 babel 版本

- v5.0.0(2015.04.02) v5 版本及以前，年代久远据说配置项不多，没有 presets
- v6.0.0(2015.10.29) 新增了 presets、性能优化、优化插件 api
- v7.0.0(2018.08.27) 经历了 3 年的一个大版本，下面我们着重介绍它

### 2.2 babel7 新特性

- 对那些已经不维护的 node 版本不予支持，包括 0.10、0.12、4、5
- Babel 团队会通过使用 “scoped” packages 的方式，来给自己的 babel package name 加上 @babel 命名空间（详情），这样以便于区分官方 package 以及 非官方 package，所以 babel-core 会变成 @babel/core
- 移除（并且停止发布）所有与 yearly 有关的 presets（preset-es2015 等）。@babel/preset-env 会取代这些 presets，这是因为 @babel/preset-env 囊括了所有 yearly presets 的功能，而且 @babel/preset-env 还具备了针对特定浏览器进行“因材施教”的能力
- 放弃 Stage presets（@babel/preset-stage-0 等），选择支持单个 proposal。相似的地方还有，会默认移除 @babel/polyfill 对 proposals 支持
- 有些 package 已经换名字：所有 TC39 proposal plugin 的名字都已经变成以 @babel/plugin-proposal 开头，替换之前的 @babel/plugin-transform。所以 @babel/plugin-transform-class-properties 变成 @babel/plugin-proposal-class-properties
- 针对一些用户会手动安装（user-facing）的 package（例如 babel-loader，@babel/cli 等），会给 @babel/core 加上 peerDependency

[详情查看： w3ctech Babel 7 于今天发布](https://www.w3ctech.com/topic/2150)

## 3 配置

大概了解了 babel，下面进入主题。

```javascript
module.exports = {
  // ...others
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader',
        exclude: '/node_modules/'
      }
    ]
  }
};
```

这是一个基本的配置，但是要用于开发还不行，需要配置 presets 和 plugs

### 3.1 presets

presets 即指定 babel-loader 按照哪个规范来编译，presets 可以像一组 babel 插件那样操作甚至可共享 options 配置

官方 presets:

- @babel/preset-env
- @babel/preset-flow
- @babel/preset-react
- @babel/preset-typescript

Stage-X (试验性 Presets),TC39 将提案分为以下阶段：

- Stage 0 - 稻草人: 只是个想法可能会有相关的 Babel 插件。
- Stage 1 - 提议: 值得深入。
- Stage 2 - 草稿: 初始规范。
- Stage 3 - 候选: 完整的规范和初始浏览器实现。
- Stage 4 - 结束: 将被添加到下一个年度版本中。

```bash
# 安装
npm install @babel/preset-env --save-dev
```

#### targets 参数

编译时会根据指定的 targets 来选择哪些语法编译哪些不编译

- targets.browsers 指定哪些浏览器
- targets.browsers: "last 2 versions" 兼容主流浏览器的最后两个版本
- targets.browsers: "> 1%" 兼容全球占有率大于 1%的浏览器
- targets.node 指定 node 版本

> 数据来自`browserslist`(一个开源项目)，和`can i use`

```javascript
module.exports = {
  // ...others
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              [
                '@babel/preset-env',
                {
                  targets: {
                    browsers: ['> 1%', 'last 2 versions'], // 指定支持哪些浏览器
                    chrome: '52' // 指定chrome版本
                  }
                }
              ]
            ]
          }
        },
        exclude: '/node_modules/'
      }
    ]
  }
};
```

### 3.2 polyfill 和 transform-runtime

> preset 只能编译新规范的语法，但是不能编译函数和方法。es6 新增的函数和方法低版本的浏览器还是不能识别，需要使用 polyfill

例如：

- Promise
- Generator
- Set
- Map
- Array.from
- Array.prototype.includes


- Babel Polyfill：会在全局定义 es6 新增的函数和方法，直接引入就能使用（会污染全局）
- transform-runtime：在局部引用，不会污染全局

```javascript
// 使用polyfill

import '@babel/polyfill';

let index = [1, 2, 3, 4].findIndex(item => {
  return item === 3;
});

console.log(index);
```

```bash
# 使用transform需要安装下面两个包
npm install --save-dev @babel/plugin-transform-runtime
npm install --save-dev @babel/runtime
```

```javascript
module.exports = {
  // ...others
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              [
                '@babel/preset-env',
                {
                  targets: { browsers: ['last 2 versions'] }
                }
              ]
            ],
            plugins: ['@babel/plugin-transform-runtime']
          }
        },
        exclude: '/node_modules/'
      }
    ]
  }
};
```
