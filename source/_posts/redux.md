---
title: Redux 入门
date: 2019-04-15 19:13:10
tags: Redux
---
## Redux 简介

> `redux`是一个数据仓库，并且，它还提供了操作（增、删、改、查）数据仓库中数据的方法。

———— 以上是我对Redux的简单理解

<!-- more -->

## 需要清楚的概念

学习和使用`redux`之前，有几个概念需要清楚：

1. `Redux`不是专门给`React`使用的（他们表面看起来好像是一套），在`React`中，照样可以是用`Vuex`，反之亦然。
2. `Redux`之所以和`React`配合使用比较多，是因为`React-redux`。
3. 在`React`中使用`Redux`不一定要使用`React-redux`。

## 先想想应该怎么用

首先，不要想着去看多少教程，先静下心来自己想一想，应该怎么用或者你自己觉得该怎么用，不然一味的的去看教程和官网之类的，大部分会被搞得晕头转向。

1. 前面已经说了，`Redux`是一个数据仓库，那么肯定就能存/取数据，所以肯定是有一个`store`（仓库）。
2. 光存数据肯定不行，肯定还需要操作数据，不然也没有意义，所以肯定有一些操作数据的方法。

下面，我们就结合我们所想的操作试试。

## 结合上面所想的开始

1. 先有一个`store`。

   ```
   import { createStore } from 'redux'; // Redux 提供的方法
   const store = createStore(fn); // fn 先别管
   ```

   上面看到，我们从`Redux`中引入了一个`createStore`的方法，并调用这个方法，就得到了一个`store`。

1. 创建了一个怎么样的`store`？

   我们创建了一个`store`,但是`Redux`并不知道我们创建的`store`是什么样的，有哪些数据，所以，我们需要给`createStore`传递一个参数，告诉我们需要什么样子的`store`（有哪些数据，有哪些操作方法等）,也就是上面的`fn`。

1. 创建一个`fn`试试

   ```const fn = (initState, action) => {}
   const fn = (initState, action) => {}
   ```

   就是上面的这样的一个函数，需要初始的数据`initState`和进行操作数据的方法`action`。

1. 可能需要多个`action`

   实际中，我们对数据可能有多个操作，比如有一个数据存储的是购物车里某个商品的数量，至少我们要有增加商品数量和减少商品数量两个方法，所以我们需要根据传入的`action`的不同来进行不同的操作，如下：

   ```
   const fn = (state = 0, action) => {
     if (action.type === 'add') {
       state = state + 1;
       return state;
     } else if (action.type === 'minus') {
       state = state - 1;
       return state;
     } else {
       return state;
     }
   };
   ```

1. 触发操作

   现在，终于创建了一个我们自己的`store`，接下来开始使用这个`store`。`Redux`提供了一个`dispatch`触发我们定义的`action`。完整代码如下：

   ```
   import { createStore } from 'redux';
   
   const fn = (state = 0, action) => {
     if (action.type === 'add') {
       state = state + 1;
       return state;
     } else if (action.type === 'minus') {
       state = state - 1;
       return state;
     } else {
       return state;
     }
   };
   
   const store = createStore(fn);
   
   let unsubscribe = store.subscribe(() =>
     console.log(store.getState()) // 每次store的数据发生改变，都会触发
   );
   
   store.dispatch({
     type: 'add'
   });
   
   store.dispatch({
     type: 'add'
   });
   store.dispatch({
     type: 'minus'
   });
   
   store.dispatch({
     type: 'minus'
   });
   
   unsubscribe(); // 取消订阅
   ```

   说明：上面有一个`subscribe`方法，是`Redux`提供的，它传入一个函数，每当`store`中的数据发生改变，`subscribe`都会执行传入的函数，并翻译一个句柄，用于取消订阅。

   从上面可以看出，`dispatch`函数传入的是一个对象，被`fn`的`action`接受，我们通过`action`的`type`来判断具体进行了怎样的动作，`Redux`的核心到这里基本就介绍完了。

   ## 再深入一点

   如果上面的内容理解了，接下来深入的部分就很容易吸收了。

   1. 我们除了传递一个`type`，也可以传入更多的参数，如：

      ```
      store.diapatch({
          type: 'type',
          value: '123'
      });
      
      // 在 action 中
      const fn = (state = 0, action) => {
        console.log(action.value); // 123
      };
      ```

      `action`传递参数在社区中有一个规范，可以参考。<https://github.com/redux-utilities/flux-standard-action>

   2. 当`action`比较多的时候，可能会出现大量的`if...ele if `等，可以使用`switch case`代替。

   3. 当有多个`fn`且没有关联的时候，通常会把拆分为多个`fn`，然后再合并，如：

      ```
      const fn1 = (state, action) => {};
      const fn2 = (state, action) => {};
      const fn3 = (state, action) => {};
      const fn4 = (state, action) => {};
      
      const rootFn = (state, action) => {
          return {
              fn1: fn1(state.fn1, action),
              fn2: fn2(state.fn2, action),
              fn3: fn3(state.fn3, action),
              fn4: fn4(state.fn4, action)
          }
      }
      ```

      上面是我们自己合并的方式，`Redux`也提供了合并的函数，示例如下：

      ```
      import { combineReducers } from 'redux';
      const fn1 = (state, action) => {};
      const fn2 = (state, action) => {};
      const fn3 = (state, action) => {};
      const fn4 = (state, action) => {};
      const rootFn = combineReducers({
        fn1,
        fn2,
        fn3,
        fn4
      })
      ```

## 在 React 中使用

其实上面的用法就可以直接使用在`React`中了，但是实际开发中，我们是结合`React-redux`来使用的，使用也很简单。具体用法可以查看我使用`Redux`和`React-redux`开发的`todo-list`。<https://github.com/xuedongwang/react-todo