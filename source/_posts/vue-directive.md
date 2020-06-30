---
title: Vue源码学习之自定义指令的生效
date: 2019-12-24 11:16:00
tags:
- vue
---

# 概述

指令（directive）是Vuejs提供的带有v-前缀的特殊特性，除了内置的v-if，v-on这些常用属性外vue.js还可以通过Vue.directive全局API创建自定义指令并获取全局指令，但它并不能让指令生效，本篇文章将介绍自定义指令是如何生效的。

<!-- more -->

# 自定义指令

虚拟DOM通过算法对比两个VNode之间的差异并更新真实的DOM节点。在更新真实的DOM节点时，有可能是创建新的节点，或者更新一个已有的节点，还有可能是删除一个节点等。虚拟DOM在渲染时，除了更新DOM内容外，还会触发钩子函数，例如在更新节点时，会触发update钩子函数，这是因为标签上通常会绑定一些指令、事件或属性，这些内容也需要在更新节点时同步被更新。因此，事件、指令、属性等相关处理逻辑只需要监听钩子函数，在钩子函数触发时执行相关处理逻辑即可实现功能。


指令的处理逻辑分别监听了create、update、destroy，其代码如下：
```javascript
export default {
  create: updateDirectives,
  update: updateDirectives,
  destroy: function unbindDirectives (vnode) {
    updateDirectives(vnode, emptyNode)
  }
}
```
虚拟DOM在触发钩子函数时，上面代码中对应的函数被执行，但无论哪个钩子函数被触发，最终都会执行一个叫updateDirectives的函数。该函数如下：
```javascript
function unbindDirectives (oldVnode, vnode) {
  if (oldVnode.data.directives || vnode.data.directives) {
    _update(oldVnode, vnode)
  }
}
```
可以看到，不论oldVnode 还是 vnode，只要其中有一个虚拟节点存在directives，那么就执行_update函数处理指令。

_update函数的代码如下：
```javascript
function _update (oldVnode, vnode) {
  const isCreate = oldVnode === emptyNode;
  const isDestroy = vnode === emptyNode;
  const oldDirs = normalizeDirectives(oldVnode.data.directives, oldVnode.context);
  const newDirs = normalizeDirectives(vnode.data.directives, vnode.context);

  const dirsWithInsert = [];
  const dirsWithPostpatch = [];

  let key, oldDir, dir;
  for (key in newDirs) {
    oldDir = oldDirs[key];
    dir = newDirs[key];
    if (!oldDir) {
      // 新指令，触发bind
      callHook(dir, 'bind', vnode, oldVnode)
      if (dir.def && dir.def.inserted) {
        dirsWithInsert.push(dir);
      }
    } else {
      // 指令已存在，触发update
      dir.oldValue = oldDir.value;
      callHook(dir, 'update', vnode, oldVnode)
      if (dir.def && dir.def.componentUpdated) {
        dirsWithPostpatch.push(dir)
      }
    }
  }
}
if (dirsWithInsert.length) {
  const callInsert = () => {
    for (let i = 0; i < dirsWithInsert.length; i++) {
      callHook(dirsWithInsert[i], 'inserted', vnode, oldVnode);
    }
  }
  if (isCreate) {
    mergeVnodeHook(vnode, 'insert', callInsert);
  } else {
    callInsert()
  }
}
if (dirsWithPostpatch.length) {
  mergeVnodeHook(vnode, 'postpatch', () => {
    for (let = 0; i < dirsWithPostpatch.length; i++) {
      callHook(dirsWithPostpatch[i], 'componentUpdated', vnode, oldVnode);
    }
  })
}
if (!isCreate) {
  for (key in oldDirs) {
    if (!newDirs[key]) {
      // 指令不再存在，触发unbind
      callHook(oldDirs[key], 'unbind', oldVnode, oldVnode, isDestroy)
    }
  }
}
```
这里先声明了6个变量 isCreate、isDestroy、oldDirs、newDirs、dirsWithInsert 与 dirsWithPostpatch，其作用如下所示。
+ isCreate： 判断虚拟节点是否是一个新创建的节点
+ isDestroy： 当新虚拟节点不存在而旧虚拟节点存在是为真。
+ oldDirs： 旧的指令集合，指oldVnode中保存的指令。
+ newDirs： 新的指令集合，指vnode中保存的指令。
+ dirsWithInsert： 其中保存需要触发inserted指令钩子函数的指令列表。
+ dirsWithPostpatch： 其中保存需要触发componentUpdated钩子函数的指令列表。
这里通过normalizeDirectives函数将模板中使用的指令从用户注册的自定义指令集合中取出来，最终取到的值为：
```javascript
{
  v-focus: {
    def: {inserted: f},
    modifiers: {},
    name: 'focus',
    rawName: 'v-focus'
  }
}
//自定义指令的代码为：
vue.directive('focus', {
  inserted: function (el) {
    el.focus();
  }
})
```
总结一下_update函数做的事情大概就是首先取到oldDirs与newDirs后，对比两个指令集合并触发对应的指令钩子函数，从循环中取到oldDirs和newDirs的指令保存到变量oldDir和dir中，然后根据判断指令的存在与否，执行对应的钩子函数。
最后，介绍下callHook函数是如何执行指令的钩子函数的，其代码如下：
```javascript
function callHook (dir, hook, vnode, oldVnode, isDestroy) {
  const fn = dir.def && dir,def[hook];
  if (fn) {
    try {
      fn(vnode.elm, dir, vnode, oldVnode. isDestroy);
    } catch (e) {
      handleError(e, vnode.context, `directive ${dir.name} ${hook} hook`)
    }
  }
}
```
该函数接受5个参数： dir、hook、vnode、oldVnode、isDestroy，他们的含义如下：
+ dir： 指令对象
+ hook： 将要触发的钩子函数名
+ vnode： 新虚拟节点
+ oldVnode： 旧虚拟节点
+ isDestroy： 当新虚拟节点不存在而旧虚拟节点存在时为真
该函数先从指令对象中取出对应的钩子函数，随后判断钩子函数是否存在。如果存在，则执行它并传递一些参数，同时使用try...catch语句捕获钩子函数在执行时可能会抛出的错误。如果抛出了错误，则调用handleError进入错误处理相关的逻辑。

这就是vue中自定义指令部分的源码，_update函数是里边比较重要的一块，我暂时也没有完全理解，希望随着深入学习后面能完全看懂！！
