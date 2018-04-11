---
title: Node.js中非I/O的异步API小结
date: 2018-04-12 01:38:08
tags:
- Node.js
- JavaScript
---

​	在提到Node.js的时候，我们会谈到异步I/O，但是Node.js中也存在一些与I/O无关的异步API，分别是setTimeout()，setInterval()，setImmediate()和process.nextTick()。

​	在了解这些之前，需要掌握一些概念：

## Event Loop（事件循环）

​	在进程启动时，Node会创建一个类似于while(true)的循环，一次循环的过程我们称为Tick。每个Tick的过程就是查看是否有事件待处理，如果有，就取出事件及其相关的回调函数。如果存在关联的回调函数，就执行它们。然后进入下个循环，如果不再有事件处理，就退出进程。
<!-- more -->

<img src="https://cdn.meishakeji.com/fontend-blog/img/node-asyc-api@01.jpg" width="300"/>

#### Event Loop的各个阶段

```
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

- timers 阶段：这个阶段执行setTimeout(callback) and setInterval(callback)预定的callback;
- I/O callbacks 阶段：执行除了 close事件的callbacks、被timers(setTimeout、setInterval等)设定的callbacks、setImmediate()设定的callbacks之外的callbacks;
- idle, prepare 阶段：仅node内部使用;
- poll 阶段：获取新的I/O事件，适当的条件下Node将阻塞在这里;
- check 阶段：执行setImmediate() 设定的callbacks;
- close callbacks 阶段：比如socket.on(‘close’, callback)的callback会在这个阶段执行。

​	每个阶段都有一个执行回调的队列，虽然每个阶段都有其特定的方式，但通常情况下，当事件循环进入给定阶段时，它将执行该阶段的所有操作，然后在该阶段的队列中执行回调，直到队列耗尽或达到回调限制，事件循环移至下一个阶段，依此类推。


#### 实现源码

```c
while (r != 0 && loop->stop_flag == 0) {
    // tick开始时更新当前时间
    uv__update_time(loop);
    // 1. 执行超时的计时器任务
    uv__run_timers(loop);
    // 2. 执行上次推迟执行的I/O回调函数
    ran_pending = uv__run_pending(loop);
    // 3. 执行idle任务
    uv__run_idle(loop);
    // 4. 执行prepare任务
    uv__run_prepare(loop);
    timeout = 0;
    if ((mode == UV_RUN_ONCE && !ran_pending) || mode == UV_RUN_DEFAULT)
        timeout = uv_backend_timeout(loop);
    // 5. 开始I/O poll, (网络I/O为epoll(Linux为例), 文件系统I/O等其他异步任务为thread pool)
    // 得到底层通知后, 通常会在本次tick执行I/O回调函数
    uv__io_poll(loop, timeout);
    // 6. 执行check任务
    uv__run_check(loop);
    uv__run_closing_handles(loop);
    // ...
    r = uv__loop_alive(loop);
    if (mode == UV_RUN_ONCE || mode == UV_RUN_NOWAIT)
        break;
}
```



## 观察者

​	在每个Tick的过程中，判断是否有事件需要处理的是**观察者**。每个事件循环有一个或多个观察者，而判断是否有事件需要处理的过程就是向这些观察者询问是否有要处理的事件。事件循环是一个典型的**生产者/消费者**模型，异步I/O、网络请求等是事件的生产者，源源不断地为Node提供不同类型的事件，这些事件被传递到对应的观察者那里，事件循环则从观察者那里取出事件并处理。



## MarcoTack、MircoTask、process.nextTick

​	在Node.js的事件循环机制中，其实有三个任务队列，**MarcoTaskQueue**、**MircoTaskQueue**和**nextTickQueue**。

​	在一个事件循环中，第一步会执行**MarcoTaskQueue**中的任务（会执行完`setImmediate`队列，而`setTimeout`或`setInterval`每个循环只且仅执行1个任务），第二步会执行**nextTickQueue**中的所有任务，最后执行**MircoTaskQueue**的所有任务。

​		**MarcoTask**一般包括script（同步代码）、`setTimeout`、`setInterval`、`setImmediate`、I/O。**MircoTask**一般包括`Promise.then`、`Object.observe`、`MutationObserver`。而**nextTickQueue**中的任务是由`process.nextTick`加入的。

​		`setTimeout`与`setInterval`的操作是追加在下轮事件循环的，`setImmediate`一定在当前循环的poll阶段完成后执行，而`process.nextTick`和**MircoTask**的操作追加在本轮事件循环，即同步任务一旦执行完成，就开始执行它们。

​	注意，`process.nextTick`不在Event Loop的任何阶段执行，而是在各个阶段切换的中间执行，在进入Event Loop时也会清空一次**nextTickQueue**。

​	下面通过一些例子来更好地理解：

```javascript
setTimeout(() => console.log(1));
setImmediate(() => console.log(2));
process.nextTick(() => console.log(3));
Promise.resolve().then(() => console.log(4));
(() => console.log(5))();
```

​	以上代码片段的执行结果是：

```
5
3
4
1
2
```

​	首先是同步代码，打印出5，接着是nextTick打印出3，之后是属于**MircoTask**的`Promise.then`，打印出4，

而到了下个事件循环，先是timers阶段的`setTimeout`，才是check阶段的`setImmediate`。

​	假设代码变为：

```javascript
setImmediate(() => console.log('immediate'));
setTimeout(() => console.log('timeout'), 0);
```

​	那么情况还真不好说，定时器执行的顺序取决于它们被调用的上下文。 如果两者都是在主模块内调用的，那么时序将受到进程性能的限制（可能会受到计算机上运行的其他应用程序的影响），因为1个事件循环需要的时间无法确定，可能长于或短于`setTimeout`的时间。

​	如果在一个I/O循环中调用：

```javascript
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

​	那么immediate一定先打印出来，这是因为`setImmediate`一定在当前循环的poll阶段完成后执行。

​	接下来是一个在《深入浅出Node.js》中的例子：

```javascript
process.nextTick(function () {
    console.log('nextTick延迟执行1');
});
process.nextTick(function () {
    console.log('nextTick延迟执行2');
});
setImmediate(function () {
    console.log('setImmediate延迟执行1');
    process.nextTick(function () {
        console.log('强势插入');
    });
});
setImmediate(function () {
    console.log('setImmediate延迟执行2');
});

console.log('正常执行');
```

​	以上代码片段在Node v8.4.0下的执行结果是：

```
正常执行
nextTick延迟执行1
nextTick延迟执行2
setImmediate延迟执行1
setImmediate延迟执行2
强势插入
```

​	而书中给出的结果是：

```
正常执行
nextTick延迟执行1
nextTick延迟执行2
setImmediate延迟执行1
强势插入
setImmediate延迟执行2
```

​	这是因为，这本书年代比较久远了，在Node v4.4.0之前，在每轮事件循环，setImmediate、setTimeout、setInterval都只能执行其中1个任务，而在Node v4.4.0的[changelog](https://github.com/nodejs/node/blob/v4.4.5/CHANGELOG.md)中，有这样一条记录：

> Updated `setImmediate` to process the full queue each turn of the event loop, instead of one per queue.

​	在每轮时间循环的check阶段，就会把`setImmediate`的队列事件全部执行。

​	因此，上面的例子，两个`setImmediate`结束后即check阶段结束后，才轮到`process.nextTick`。

## 小结

​	Node.js中非I/O的异步API之间的相互关系其实非常复杂，网络上的文章基于各种版本的Node，年代久远、众说纷纭，本文总结的是在Node v8.4.0下的情况，具体还需要参阅[官方文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)，如有错漏，欢迎指正。