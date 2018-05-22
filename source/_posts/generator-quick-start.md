---
title: Generator函数初接触及应用
date: 2018-05-222 20:00:00
tags: 
- js
---

异步编程是js里面很重要的一个概念，Generator是ES6提出的新的异步编程函数，它允许程序在执行过程中暂停，执行别的工作，然后再返回暂停的地方重新执行。

## Generator基本概念
Generator函数的形式跟普通函数有下面两个区别
> 1. `function`和函数名之间有一个*号
> 2. 函数内部可以使用`yield`语句，定义不同的状态


<!-- more -->
```js
function* helloWorld() {
    yield 'hello';
    yield 'world';
    return 'ending';
}
var hw = helloWorld()
```
直接调用Generator函数并不会马上执行，只有当主动调用`next()`的时候，该函数内部语句才会执行，
`yield`语句相当于函数的中断点，当遇到`yield`语句的时候Generator函数会中断执行，保留现场，把函数执行权交出，等到下一次主动调用`next()`的时候，Generator才会恢复执行并从上一次中断的位置继续执行下去
`Generator`函数内部允许没有`yield`语句，这时候可以把它充当一个暂缓执行的函数使用，只用调用next()才执行
`yield`语句返回一个对象，包含`value`和`done`两个key，`value`是本次`yield`的返回值，如果没有就返回`undefined`，`done`表示后面是否还有`yield`中断点，即该Generator是否已经执行完毕
```js
hw.next() // 函数中断执行，返回{ value: 'hello', done: false }
hw.next() // 函数中断执行，返回{ value: 'world', done: false }
hw.next() // 函数执行完毕，返回{ value: 'ending', done: true }
```
在Generator中主动调用return会终止整个Generator函数

## Generator应用
### 异步函数，同步方式表达
比如很常见的一个场景，异步请求数据并渲染页面，使用Generator可以是函数看起来跟同步函数并无区别，只是多了一个`yield`， 把`yield`去掉，看起来就是一个普通函数，逻辑更加清晰
```js
function* render() {
    showLoading()
    const res = yield getData();
    console.log(res)
    hideLoading()
}
function getData() {
    $.ajax().done(function(res){
        r.next(res)
    })
}
const r = render()
r.next()
```
### 流程管理
当存在多个流程需要按顺序执行时，使用回调函数方式很容易使代码混乱，逻辑难懂；使用Promise链式调用方式会使代码冗余，每个步骤返回都被Promise包装了，看去都是`then`；而使用Generator函数相对而言，更加直观，逻辑也更加清晰
```js
// 回调函数方式
function step() {
    function step1(res1) {
        function step2(res2) {
            function step3(res3) {
                // do something
            }
        }
    }
}

// promise方式
step()
    .then(res1 => Promise.resolve())
    .then(res2 => Promise.resolve())
    .then(res3 => {
        // do something
    })
    .catch(err => {})
   
// Generator方式
function* step() {
    yield step1()
    yield step2()
    yield step2()
    // do something
}
```
### 自动流程管理
接上面的例子，因为Generator充当的是一个异步操作的容器，使用Generator来进行流程管理的时候要手动调用`next()`才能继续执行下一个流程，如果要需要自动调用`next()`，一般有两种方式：
> 1. 回调函数方式。把异步操作包装成`Thunk`函数，也就是在每一次回调里自动调用`next()`，直到`yield`返回`done`为`true`，结束调用
> 2. Promise方式。将异步操作包装成 Promise 对象，在回调函数中中调用`next()`，直到`yield`返回`done`为`true`，结束调用

Promise方式实现自动流程管理
```js
function asyncStep() {
    return Promise.resolve()
}

function* fn(){
    yield asyncStep()
    yield asyncStep()
    yield asyncStep()
};

function autoRun(gen){
    const g = gen()
    // 回调自身，直到done为true
    function next(){
      const result = g.next()
      if (result.done) return result.value
      result.value.then(() => next())
    }

    next()
  }
  
  autoRun(fn)
```