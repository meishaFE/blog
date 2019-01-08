---
title: 初识 Virtual DOM
date: 2018-11-05 22:56:44
tags:
- 虚拟DOM
- JavaScript
---

## 前言
Virtual DOM（以下简称 vDom）想必大家都不陌生，用过 React 和 Vue 框架的应该都听说过，但它到底是个什么东西呢，今天就让我们来了解一下吧。

<!-- more -->

## 为什么需要 vDom
首先我们要知道 DOM 操作为什么会慢：
- 在浏览器中 DOM 的实现和 js 的实现是分离的，我们通过 js 调用操作 DOM 的接口，相当于两个独立的模块发生了交互，这种跨模块的调用是很耗性能的，就像你在两座岛之间来往肯定没有比在一座岛之间快一样
- 它导致了浏览器的重绘（repaint）和重排（reflow）
- DOM 本身的附加属性很多（标准就是这样规定的）

既然 DOM 操作这么昂贵，那如何进行优化呢？很简单，就是减少 DOM 操作。DOM 操作很容易影响页面的性能，但操作 js 远比操作 DOM 快得多，所以就有了 vDom 的出现。

## 什么是 vDom
vDom 其实就是一个 js 对象，我们可以通过 js 来表示一个真实 DOM。

```js
// 这是真实DOM
<ul id='app' class='list'>
  <li>这是第一条</li>
  <li>这是第二条</li>
</ul> 
```
```js
// 这就是虚拟DOM
let vDom = {
  tagName: 'ul',
  attributes: {
    id: 'app',
    class: 'list'
  },
  children: [
    {
      tag: 'li',
      text: '这是第一条'
    },
    {
      tag: 'li',
      text: '这是第二条'
    }
  ]
}
```
如上述代码所示，用 js 来表示一个 DOM 节点是很简单的事情，而一个真实的 DOM 有几十个属性，相对来说 vDom 就少得多，因为它只模拟了真实 DOM 的部分属性，比如它的节点类型、样式属性和子节点等。

## 从 vDom 如何转变成为真实 DOM
现在上面的 ul 只是一个 js 对象表示的 DOM 结构，它并不存在于页面中。所以接下来我们需要根据这个 js 对象在页面中添加真实的 ul 元素。
那如何添加呢？其实就是用 `document.createElement('ul')` 来创建 ul 标签，这里我们把它封装成一个 render 函数，render 方法会根据 tagName 来构建一个真正的 DOM 节点，然后设置这个节点的属性，最后递归地把自己的子节点也构建起来。具体代码如下：

```js
Element.prototype.render = function () {
  var el = document.createElement(this.tagName) // 创建标签
  var attrs = this.attributes
 
  for (var attrName in attrs) { // 设置节点属性
    var attrValue = attrs[attrName]
    el.setAttribute(attrName, attrValue)
  }
 
  var children = this.children || []
 
  children.forEach(function (child) {
    var childEle = (child instanceof Element)
      ? child.render() // 如果子节点也是vDom，递归调用
      : document.createTextNode(child) // 如果字符串，只构建文本节点
    el.appendChild(childEle)
  })
  return el
}
```
至此，我们就完成了从创建 vDom 到将其渲染成真实 DOM 的工作了。

## diff 算法
这部分可能有点跳跃，因为这是 vDom 的一个精髓部分。
既然我们可以用 js 对象来表示 DOM 结构，那么当数据状态发生改变时，一棵新的 vDom 就会生成，然后和旧的 vDom 进行比较，并且把变化的那部分进行实际 DOM 操作，以最小的代价来更新必要的DOM，这显然大大提高了操作效率。
比如我们要添加一个子节点，可以用 `vDom.children.push({tag: 'li', text: '这是第三条'});` 来更新 vDom，而不是直接在代码中直接调用像 `document.getElementById().appendChild()` 之类的 DOM 方法，操作 vDom 就会像改变 js 对象那样的省时省力，毕竟 vDom 存在于内存中。
那难点在于什么呢？就是如何比较两棵 vDom 的差异啦，也就是 diff 算法！不过这里先不深究啦（主要是没有两把刷子），就先讲个大概，让大家有点印象就行。
diff 算法初识：也就是比较两棵树的差异。两个树的完全的 diff 算法是一个时间复杂度为 O(n^3) 的问题，但是在前端当中，我们一般很少会跨层级地移动 DOM 元素，所以采用同级对比的方式。简言之 vDom 只会对同一个层级的元素进行对比，比如上面的 ul 只会和同一层级的 ul 对比，第二层级的 li 只会跟第二层级的 li 对比，如果节点类型不同就直接取代掉旧的节点，而不是继续比较；如果节点类型相同就会对这个节点进行修改，而不是严格的比较所有属性是否相同，这样算法复杂度就可以达到 O(n)。同时，每个有差异的节点都会被写成一个对象汇总起来，最后要做的就是把差异渲染到真正的 DOM 上即可。
```js
let patches = diff(tree, newTree) // diff算法先请自行google，这里先略过
let patches = { // 这是diff算法输出的结果，大概长这样
  // 1 和 5 是节点的遍历顺序，type对应的是改动类型
  1: [{type: REPLACE, node: newNode}],
  5: [{type: TEXT, content: '文本内容改变'}, {type: PROPS, props: {class: "new-list"}]
}
patch(root, patches) // 把差异渲染到真实DOM上
```

Vue 在2.0版本引入了 vDom。它的 vDom 是在 snabbdom 库的基础上修改的。snabbdom 是一个开源的 vDom 库。它主要是通过`h()`来将模板解析成 vDom，如果是首次渲染，则通过`patch(container, vDom)`将 vDom 渲染至页面，如果是二次渲染，先通过diff算法比较新旧 vDom 的差异`let patches = diff(vDom, newVDom)`，然后以最小的代价渲染页面`patch(root, patches)`，有兴趣的小伙伴们可以去看看 snabbdom 的源码。

## 缺点

- 代码更多，体积更大
- 内存占用增大，vDom 会保存在内存中
- 简单、少量的 DOM 操作使用 vDom 的成本反而会更高，但对于大多数的单页面应用来说，它是会带来提升的

## 一句话小结
vDom 的本质是个 js 对象, 它可以用来表征真实 DOM，它的主要目的是就是减少 DOM 操作次数。