---
title: 监听滚动，你还在用window.addEventListener('scroll')?
date: 2019-9-16 15:45:55
tags:
- 前端API
---

## vue-lazyload

`vue-lazyload`这个组件相信大家在项目里应该都用到过，也都挺熟悉的；这天闲来无事，便想着去看看它是如何实现的；之前一直只大概知晓原理，即每次页面加载时只去加载视窗内的资源，其他未滚动到视窗内的资源则暂时先用默认样式来代替，以达到资源按需加载，让加载的速度也变得更快的目的。

稍微推测了下，按照这个原理的话，代码里一定会要去监听页面的滚动，并且计算当前视窗和dom元素的位置，而且滚动是一件非常频繁的事件，应该大概率会用到节流函数和`getBoundingClientRect()`方法；果不其然，源码`lazy.js`里果然有引入`throttle`这个玩意；猜测是对的，可是继续看下去，看到了一个之前从未接触到的东东

<!-- more -->

## 进入主题

对的，在源码里有`_initIntersectionObserver`这样一个函数，而函数的具体实现也挺简单，如下：
``` javascript
/**
  * init IntersectionObserver
  * set mode to observer
  * @return
  */
  _initIntersectionObserver () {
    if (!hasIntersectionObserver) return
    this._observer = new IntersectionObserver(this._observerHandler.bind(this), this.options.observerOptions)
    if (this.ListenerQueue.length) {
      this.ListenerQueue.forEach(listener => {
        this._observer.observe(listener.el)
      })
    }
  }

  /**
    * init IntersectionObserver
    * @return
    */
    _observerHandler (entries, observer) {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.ListenerQueue.forEach(listener => {
            if (listener.el === entry.target) {
              if (listener.state.loaded) return this._observer.unobserve(listener.el)
              listener.load()
            }
          })
        }
      })
    }

```

看到这里，心里就有疑问了，因为`IntersectionObserver`这个玩意之前没碰到过，所以，今天我的主题不是给大家分析`vue-lazyload`这个组件的源码，而是`IntersectionObserver`。碰到不知道的东西怎么办呢？当然是左转问百度，右转问谷歌啦。既然，源码里可以直接使用`new` 关键字，而搜遍了整个源码里也没得这个构造函数，则证明它是一个内置的接口。必应上一搜索，果不其然,MDN官方文档给出的文档说明如下：
``` html
IntersectionObserver接口 (从属于Intersection Observer API) 提供了一种异步观察目标元素与其祖先元素或顶级文档视窗(viewport)交叉状态的方法。祖先元素与视窗(viewport)被称为根(root)。

当一个IntersectionObserver对象被创建时，其被配置为监听根中一段给定比例的可见区域。一旦IntersectionObserver被创建，则无法更改其配置，所以一个给定的观察者对象只能用来监听可见区域的特定变化值；然而，你可以在同一个观察者对象中配置监听多个目标元素。

``` 

通读下来，对这个接口有了一个模糊的概念，简单的理解就是监听一个目标元素与视窗可见区域的可见比例的变化（即用户能不能看到这个目标元素）；继续往下看它的用法，直接上官方示例（无限滚动）：

``` javascript
var intersectionObserver = new IntersectionObserver(function(entries) {
  // If intersectionRatio is 0, the target is out of view
  // and we do not need to do anything.
  if (entries[0].intersectionRatio <= 0) return;
  loadItems(10);
  console.log('Loaded new items');
});
intersectionObserver.observe(document.querySelector('.scrollerFooter')); // 开始观察

```

以上，使用起来还是挺简单的，实例化构造函数`IntersectionObserver`后返回一个观察器实例`intersectionObserver`，该构造函数里传入了一个参数`callback`,该参数`callback`回调函数里的参数`entries`是一个`IntersectionObserverEntry`对象的数组，该`IntersectionObserverEntry`对象提供了目标元素的一些信息，一共有六个属性：

``` JSON
  {
    time: 3567.92, // 可见性发生变化的时间，是一个高精度时间戳，单位为毫秒
    rootBounds: // 返回一个 DOMRectReadOnly 用来描述交叉区域观察者(intersection observer)中的根.
      ClientRect {
        bottom: 920,
        height: 1024,
        left: 0,
        right: 1024,
        top: 0,
        width: 920
      },
      boundingClientRect: // 返回包含目标元素的边界信息的DOMRectReadOnly. 边界的计算方式与  Element.getBoundingClientRect() 相同.
        ClientRect {
        // ...
        },
      intersectionRect:  // 返回一个 DOMRectReadOnly 用来描述根和目标元素的相交区域.
        ClientRect {
        // ...
        },
      intersectionRatio: 0.62, // 返回intersectionRect 与 boundingClientRect 的比例值
      target: element // 被观察的目标元素，是一个 DOM 节点对象
  }
```

以上一大堆信息中，用的最多的应该就是`intersectionRatio`这个属性了，使用它我们可以知道当前监听元素在可是窗口所占的比例，从而监听目标元素位置。

此外，观察期实例`intersectionObserver` 有以下方法

| 方法名        | 说明           | 参数    | 返回值                                 |
| ----------- | -------------- | ------- | ------------------------------------ |
| disconnect  | 终止对所有目标元素可见性变化的观察 | None  | undefined |
| observe  | 向IntersectionObserver对象监听的目标集合添加一个元素，可同时监听多个目标元素 | targetElement:目标监听元素（此元素必须是根元素的后代）  | undefined |
| takeRecords  | 返回一个 IntersectionObserverEntry 对象数组 | None  | IntersectionObserverEntry 对象数组, 每个对象包含目标元素与根每次的相交信息 |
| unobserve  | 停止对一个元素的观察 | 要取消观察的目标，如果没有提供，方法不做任何事情，也不会抛出异常。  | undefined |

## 应用

了解了这些api的使用方法后，基本上是可以集成`IntersectionObserver`这个接口到我们日常的开发中了，目前我能想到的有
- 惰性加载

- 无限滚动

- 自动吸附

使用此接口来实现以上效果就会变得更加容易，且效率更高

## 兼容性处理

当然，每次用到这种比较少见api时，心里难免会心慌慌的，因为既然没有大规模使用，那肯定是有它的兼容性存在的。屁颠屁颠的跑到`can i use`上查询了一番，IE全军覆没，所幸的是Edge从16后开始慢慢兼容，Firefox和Chrome在新版本里也都开始支持了；移动端方面IOS Safari从12.3才开始支持，Android从版本76后开始支持；整体看来，它的支持率在慢慢上升，说不定哪天也会成为行业标准，但是，为了程序健壮性考虑，还是得对它的兼容性作一定的处理；

``` javascript
 // 兼容判断

const inBrowser = typeof window !== 'undefined'

function checkIntersectionObserver () {
  if (inBrowser &&
    'IntersectionObserver' in window &&
    'IntersectionObserverEntry' in window &&
    'intersectionRatio' in window.IntersectionObserverEntry.prototype) {
  // Minimal polyfill for Edge 15's lack of `isIntersecting`
  // See: https://github.com/w3c/IntersectionObserver/issues/211
    if (!('isIntersecting' in window.IntersectionObserverEntry.prototype)) {
      Object.defineProperty(window.IntersectionObserverEntry.prototype,
        'isIntersecting', {
          get: function () {
            return this.intersectionRatio > 0
          }
        })
    }
    return true
  }
  return false
}
```


