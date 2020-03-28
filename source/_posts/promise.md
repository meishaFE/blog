---
title: '渐进增强的 Promise'
date: 2020-03-28 13:06:03
tags:
---

## 前言
最近这段时间由于疫情的原因，在家实在闷得慌，所以看了下 js 的一些基础知识，从前不是很了解的 Promise 突然豁然开朗了不少，于是就赶紧趁热打铁写下来（这就是温故而知新的感觉吗，哈哈哈😁）。
<!-- more -->
> 确实是待久了，🌸樱花🌸都开了。

## 为什么要用 Promise
- 一个很显然的原因就是它的链式调用能够解决回调地狱带来的一些问题（不利于阅读与维护、不好调试等），这点想必大家都清楚。
- 其实 Promise 和传统的回调还有一个细微的区别，就是回调函数的定义时机，传统的回调我们是得先有回调函数再执行（因为你要把函数当作参数传，得先写好），而 Promise 则更加灵活，它是先执行函数体，然后在你需要 then 的时候再去写回调函数也不迟，不知道说清楚没有，大家可以细品一下😂。

## 基础用法
要想知道某个东西怎么写的，就要先学会用，所以读者大大们如果还有没用过的话，赶紧去学下再来看吧，因为本文的目标是手写一个 Promise 以及 catch、finally、all、race、resolve 等附加方法，下面是最简单的用法展示👇：
```js
let p = new Promise((resolve, reject) => { // 这里面的代码是同步执行的
  resolve(1)
}).then(res => { // 这里面的代码是异步的
  console.log('成功', res)
}, err => {
  console.log('错误', err)
})
// 成功 1
```

## 知识前置
事实上 Promise 是有个 Promises/A+ 规范的（这东西相当于一个需求文档），这里我会先罗列几个必知必会的点，毕竟要手写嘛✍，肚子里得有点墨水：
- 我们要知道一个 Promise 有三种状态：pending，success，fail，并且状态只能从 pending 到 success 或者从 pending 到 fail，并且只能改变一次，是不可逆的（正经点的状态名称：pending、fulfilled、rejected，当然这不重要）
- resolve 和 reject 是用来改变 Promise 自身状态和值的，并触发后面 then 里面的回调
- then 里面的参数如果不是函数将被忽略，具有穿透效果，这个后面会细说
- new Promise() 返回一个 promise 对象，每个 then 返回一个新的 Promise，因为 Promise 的状态只允许被改变一次，所以每次都得返回新的

下面的描述我也没写的规范那么正经，因为我有时候总觉得那样不利于理解，一堆英文名字就把你给愣住了😯。

## 开始手写
首先我们把前面提到的基础用法简化下：
```js
let p = new Promise(fn).then(fn1, fn2)
```
这样一来上面的结构就明了许多，既然需要 new，那么它首先是个构造函数，接收一个函数参数 fn，然后实例有个 then 方法，接收两个参数 fn1 和 fn2，也都是函数。此外由知识前置里面的内容可知 Promise 里面自身得维护一个状态，所以我们可以先写出一个大体框架：
```js
class MyPromise {
  constructor(fn) {
    this.status = 'pending' // 保存状态
    this.successValue = null // 保存成功的值
    this.failValue = null // 保存失败的值
    fn() // 因为 Promise 里面的代码是同步执行的，所以直接进来需要直接调用
  }
  then(successFn, failFn) {}
}
```
我们知道在执行 fn 的时候其实还有两个参数，就是 fn(resolve, reject)，所以我们需要完善它，在 MyPromise 里面定义 resolve 和 reject 这两个函数，当外界调用 resolve 和 reject 时就是改变 MyPromise 里面的状态和值，并触发相应的回调函数，就像下面这样：
```js
constructor(fn) {
    this.status = 'pending' // 保存状态
    this.successValue = null // 保存成功的值
    this.failValue = null // 保存失败的值

    let resolve = (successValue) => { // 这个 successValue 是外部调用传进来的值
      this.status = 'success'
      this.successValue = successValue
    }
    let reject = (failValue) => { // 这个 failValue 是外部调用传进来的值
      this.status = 'fail'
      this.failValue = failValue
    }

    fn(resolve, reject)
}
```

## then
constructor 里面的东西大概写完了，接下来我们简要写下 then 方法，then 方法里面不是有两个函数参数吗，根据 status 的状态执行其中一个即可，就像下面这样：
```js
then(successFn, failFn) {
    if (this.status === 'success') {
        successFn(this.successValue)
    } else if (this.status === 'fail') {
        failFn(this.failValue)
    }
}
```
ok，我们来测试一下：
```js
let p = new MyPromise((resolve, reject) => {
  console.log('1')
  resolve(100)
}).then(res => {
  console.log('2')
  console.log('成功', res)
}, err => {
  console.log('错误', err)
})
console.log('3')
// 1
// 2
// 成功 100
// 3
```
不错不错，可以成功打印出 100，但是有个问题，`console.log` 的顺序应该是 1、3、2，因为 then 里面的内容是异步执行的，所以我们需要 setTimeout 来简单模拟下，把 then 操作延后，就像下面这样：
```js
then(successFn, failFn) {
    if (this.status === 'success') {
      setTimeout(() => {
        successFn(this.successValue)
      })
    } else if (this.status === 'fail') {
      setTimeout(() => {
        failFn(this.failValue)
      })
    }
}
```
不错不错，看起来好像可以了，但是如果我这样写呢：
```js
let p = new MyPromise((resolve, reject) => { // 连续调用
    resolve(100)
    reject(-1)
}).then(res => {
  console.log('成功', res)
}, err => {
  console.log('错误', err)
})
// 错误 -1
```
上面结果输出 -1 当然是错的，因为连续调用 resolve 或 reject 是无效的，Promise 只允许被改变一次，所以我们需要加个限制条件：
```js
let resolve = (successValue) => {
  if (this.status !== 'pending') return // 状态已经改变过就不往下执行了
  this.status = 'success'
  this.successValue = successValue
}
let reject = (failValue) => {
  if (this.status !== 'pending') return // 状态已经改变过就不往下执行了
  this.status = 'fail'
  this.failValue = failValue
}
```
当然还没完，一个新的问题诞生了，如果我这样写呢：
```js
let p = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve(100)
  }, 2000);
}).then(res => {
  console.log(res)
}, err => {
  console.log(err)
})
```
没有输出任何东西，这是因为当我调用 then 的时候，MyPromise 里面的东西还没执行完，状态未被改变，所以 then 里面的成功和失败都不会调用，因此我们需要在 then 中加上一种情况，就是如果 MyPromise 里面还没执行完就先把 then 中的 fn1 和 fn2 放进数组中存起来，等到 MyPromise 里面执行完再把数组拿出来遍历执行（当然也要放进 setTimeout 中），就像下面这样（接下来都用图了😁）：

![](/images/promise/1.png)

事实上你可以把 resolve 或 reject 当成触发 then 里面函数的开关，只要碰到 resolve 或 reject，与之对应的 then 里面的函数也该执行了（放入微任务执行，我们用的是宏任务替代，关于事件循环也是个挺有意思的知识点，大家可以自己去了解一下）。一句话说就是 resolve 和 reject 的作用就是在合适的时间点执行 then 里面的回调函数，有点观察者模式的意思（你品，你细品🤔）。

## then 加强
写到这里，其实还有个大问题，前面说过了，then 是有返回的，并且是个新的 Promise，还支持链式调用，显然我们的并不具备这样的功能，所以现在我们需要完善 then，主要思想就是在 then 里面套一层 Promise，此外你要知道如果 then(fn1, fn2) 继续向下传递的话，传递的值是 fn1 或 fn2 的返回值，看下面这张图你可能会清晰点：

![](/images/promise/2.png)

👌，理清了这个东西之后我们就来看下具体是怎么改的：

![](/images/promise/3.png)

细心点你会发现这里 then 里面函数的返回值还可能是个 Promise，所以我们需要对 then 里面函数的返回值做个判断，如果返回值是个 Promise 就需要等这个 Promise 执行完再调用 resolve 和 reject，就像下面这样：

![](/images/promise/4.png)

执行一下下面的测试代码：
```js
let p = new MyPromise((resolve, reject) => {
  resolve(100)
}).then(res => {
  console.log('成功', res)
  // return 0
  return new MyPromise((resolve2, reject2) => {
    setTimeout(() => {
      resolve2(0)
    }, 1000)
  })
}, err => {
  console.log('错误', err)
}).then(res2 => {
  console.log('成功2', res2)
}, err2 => {
  console.log('错误2', err2)
})
// 成功   100
// 成功2  0 (1s后打印出来)
```
写到这里，then 里面的东西已经写的差不多了，但其实还是有问题的，我们这里只是解决了一层 Promise 的嵌套，如果你多嵌套几个 Promise 就不行了，这个需要我们把上面公共的部分提取出来然后递归调用，写起来不复杂，但会有点绕容易晕，此外我们也没有对循环调用同一个 Promsie 做判断以及一些异常捕获，因为我们理解到这里就差不多了👏。当然了，我会在文末附上完整的代码😬，里面也有详细的注释。

## 穿透
什么是穿透呢？让我们来看下面的代码：
```js
let p = new Promise((resolve, reject) => {
  resolve(100)
}).then()
.then(1)
.then(res => {
  console.log('成功', res)
}, err => {
  console.log('错误', err)
})
// 成功 100
```
简单来说就是，如果 then 中的参数不是函数或为空，then 之前的值还能够继续向下传递，其实这个写起来很简单，就是在 then 里面的一开始判断下参数是否为函数，不是的话就包装成函数，并把之前的值当作返回值，具体操作如下：

![](/images/promise/5.png)

注意我们这里抛出错误 throw 不能写成这样 `return new Error('xxx')`，因为这不是抛出错误，是返回一个错误对象，等价于 `return {}`，它是个正常的返回值，而不是错误，好好体会一下。

## catch
catch 这东西其实和 then 一毛一样，只不过不需要成功回调，promise.catch(fn) 相当于 promise.then(null, fn)，也就是说如果 catch 后面如果还有 then 也是可以继续执行的，我们直接看下面的代码就了解了：

![](/images/promise/6.png)

## Promise.resolve 和 Promise.reject
这里我们先简要看一下 Promise.resolve 的用法：
```js
Promise.resolve(1).then(res => console.log(res))
Promise.resolve(
    new Promise((resolve, reject) => resolve(2))
).then(res => console.log(res))
// 1
// 2
```
首先 Promise.resolve 是个静态方法（就是只能用类来调用，好比 `Math.random()`），它可以进行链式调用，所以它返回的也是个 Promise，只不过要注意的是 resolve 里面接收的参数可以是 Promise 和一般值（数字、字符串、对象等），如果是 Promise 则需要等这个参数 Promise 执行完再返回，让我们看下下面的代码：

![](/images/promise/7.png)

Promise.reject 也是一样的写法，它们都算是语法糖，这里就略过了。

## finally
finally 的特点是不论正确与否，它总会执行，接收一个函数参数，并且返回 Promise，不过要注意的是它向下传递的值是上一次的值而不是 finally 中的值，具体如下：

![](/images/promise/8.png)

## Promise.all
还是一样我们先来看下具体用法：
```js
MyPromise.all([new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(1)
  }, 1000);
}), 2, 3]).then(res => {
  console.log(res)
})
// [1, 2, 3]    (1s后打印)
```
首先 all 方法接收一个数组参数，数组的每一项可以是 Promise，也可以是常量等有返回值的东西；其次使用 all 方法之后也可以用 then 进行链式调用，所以 all 方法返回的也是个 Promise；最后 all 是并发执行，其实就是写个循环，不过只有全部成功才算是成功，否则就算失败，直接看下面的代码：

![](/images/promise/9.png)

## Promise.race
race 其实和上面的 all 差不多，但是规则有点不一样，它也是接收一个数组，只不过返回的就一项，最先返回成功就成功，最先失败就失败，代码也是和上面雷同，具体如下：

![](/images/promise/10.png)

## 小结
所谓最好的输入就是输出，一晃又是几个月没写文章了，所以特此沉淀，还是心虚啊。希望本篇文章能够对你有所帮助，不知道写的清不清楚😁。最后祝福大家百毒不侵，回见👋。

ps：[Gituhub 代码地址](https://github.com/lgq627628/2020/blob/master/%E6%89%8B%E5%86%99%E5%8E%9F%E7%94%9F/%E5%8E%9F%E7%94%9F%E6%96%B9%E6%B3%95/promise/promise7.js)
