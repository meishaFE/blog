---
title: Vue源码之nextTick详解
date: 2019-11-11 10:00:00
tags:
- vue
- nextTick
---

# nextTick详解

最近在看vue源码的时候，看到了nextTick的定义，通读了一番，正所谓最好的输入就是输出，于是写下这篇博客，同时也希望能够给到别人帮助。

<!-- more -->

话不多说，由于代码较少，直接上源码。  
```javascript
import { noop } from 'shared/util'
import { handleError } from './error'
import { isIE, isIOS, isNative } from './env'

export let isUsingMicroTask = false

const callbacks = []
let pending = false

function flushCallbacks () {
	pending = false
	const copies = callbacks.slice(0)
	callbacks.length = 0
	for (let i = 0; i < copies.length; i++) {
		copies[i]()
	}
}

let timerFunc

if (typeof Promise !== 'undefined' && isNative(Promise)) {
	const p = Promise.resolve()
	timerFunc = () => {
		p.then(flushCallbacks)
		if (isIOS) setTimeout(noop)
	}
	isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
	isNative(MutationObserver) ||
	// PhantomJS and iOS 7.x
	MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
	// Use MutationObserver where native Promise is not available,
	// e.g. PhantomJS, iOS7, Android 4.4
	// (#6466 MutationObserver is unreliable in IE11)
	let counter = 1
	const observer = new MutationObserver(flushCallbacks)
	const textNode = document.createTextNode(String(counter))
	observer.observe(textNode, {
		characterData: true
	})
	timerFunc = () => {
		counter = (counter + 1) % 2
		textNode.data = String(counter)
	}
	isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
	timerFunc = () => {
		setImmediate(flushCallbacks)
	}
} else {
	timerFunc = () => {
		setTimeout(flushCallbacks, 0)
	}
}

export function nextTick (cb?: Function, ctx?: Object) {
	let _resolve
	callbacks.push(() => {
		if (cb) {
			try {
				cb.call(ctx)
			} catch (e) {
				handleError(e, ctx, 'nextTick')
			}
		} else if (_resolve) {
			_resolve(ctx)
		}
	})
	if (!pending) {
		pending = true
		timerFunc()
	}

	if (!cb && typeof Promise !== 'undefined') {
		return new Promise(resolve => {
			_resolve = resolve
		})
	}
}
```

上面的代码便是nexTick实现的完整代码，实现过程并不复杂，我会将代码从头到尾进行梳理解析，帮助大家理解。

## 引入的功能
在代码里共引入了`noop handleError isIE isIOS isNative`这五个功能。其中的`isIE isIOS handleError`功能一看就很清晰，不过其中`handleError`的处理比较繁琐一点。  

当你调用这个功能的时候，它会调用父级的`errorCaptured`钩子事件，如果父级返回的不是`false`，那么便再向上一级传播，直到root为止，有点像冒泡事件。
具体详细规则可以看[官方说明](https://cn.vuejs.org/v2/api/#errorCaptured)。  

`noop`则是一个什么也不干的空函数。

`isNative`正如起名，就是用来判断一个js内建函数（如`Array`）是否是原生代码的实现。之前不知道还能这样来判断，算是又涨了见识。具体实现如下，比较简单，就是用toString的值来进行判断。
```javascript
export function isNative (Ctor) {
  return typeof Ctor === 'function' && /native code/.test(Ctor.toString())
}
```

这样一来，引入的功能就都清晰是做什么的了，接下来我们再来看看每个变量与函数的定义和具体的值。

## 变量与函数的定义与具体赋值
其中共定义了`isUsingMicroTask
callbacks
pending`三个变量，都是全局变量（当前函数执行环境）。  

`isUsingMicroTask`指的是是否使用微任务的方法来将我们传入的方法进行调用。因为实际浏览器的情况比较复杂，所以下面有专门的函数`timerFunc`进行判断，函数我们稍后再讲。  

`callbacks`是一个方法数组，在执行微任务队列前，每调用`nextTick`传入的方法便保存在这个数组里面。

`pending`用来判断处于执行中的变量。

函数有两个，分别是`flushCallbacks timerFunc`。

`flushCallbacks`的作用是清空任务队列。代码比较简单。我们从里面摘出来看看。
```javascript
function flushCallbacks () {
	pending = false
	const copies = callbacks.slice(0)
	callbacks.length = 0
	for (let i = 0; i < copies.length; i++) {
		copies[i]()
	}
}
```
可以看到是将`pending`设置为false，意味着已执行完清空队列的操作， 不处于等待状态，可以执行下一次清空队列操作。同时需要注意的是这里采取的方式不是直接对数组进行遍历操作,而是先放到另外一个数组后再进行遍历操作。这样处理的原因我只能想到是因为在函数执行的期间可能报错，导致后面的代码不执行，从而无法清空掉任务队列。所以先采取清空队列，再由另一个复制队列去执行函数。  

`timerFunc`则是用来调用`flushCallbacks`的方法，正是由这个方法来决定是否使用微任务并且是哪种方法来使用`nextTick`。作者在这里写了一大段的注释，简而言之就是目前的策略是优先采用微任务，但是微任务还是存在一些不可避免的问题存在。而实现微任务有两种途径：`Promise MutationObserver`，优先采用`Promise`。

```javascript
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    // In problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
}
```

第一个进行的判断是是否使用`Promise`。可以看到是检测是否有原生的`Promise`存在。然后对`timerFunc`进行赋值，就变成每次调用这个方法的时候是执行直接`then`方法，因为是已经处于`resolve`状态了，所以相当于`then`方法会直接触发，所以就进入了微任务队列。在这个地方还对IOS进行了判断，如果是IOS的环境则使用`setTimeout`执行一个空函数。原因作者有写到是因为在UIWebViews中会出现莫名的卡顿，所以在这里强行生成一个宏任务，使得微任务也随之执行。

```javascript
else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  // PhantomJS and iOS 7.x
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  // Use MutationObserver where native Promise is not available,
  // e.g. PhantomJS, iOS7, Android 4.4
  // (#6466 MutationObserver is unreliable in IE11)
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} 
```

如果`Promise`无法使用的话，那么第二个进行的判断是是否有原生的`MutationObserver`，并且其中还对单独的一个特殊情况做了处理（提示中还提到了IE11中`MutationObserver`是不可靠的）。

首先创建了一个变量`counter`，这个变量只是作为一个不断变化的值来触发`MutationObserver`的监听。然后建立了一个监听对象`observer`，回调传入的则是`flushCallbacks`，意思是如果监听触发则触发`flushCallbacks`这个函数。然后创建了一个文本节点`textNode`，然后让刚刚建立的监听对象来监听这个文本节点的变化。于是设置`timeFunc`函数的功能为修改文本节点的值，从而触发监听的回调事件，也就是`flushCallbacks`。

```javascript
else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // Fallback to setImmediate.
  // Technically it leverages the (macro) task queue,
  // but it is still a better choice than setTimeout.
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  // Fallback to setTimeout.
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```

如果不存在`MutationObserver`的话，那么就是先判断是否存在原生的`setImmediate`，如果都没有，就使用`setTimeout`函数。


## nextTick函数详解
最后我们来看`nextTick`这个函数的具体实现。
```javascript
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```
其实也不复杂，首先是在`callbacks`队列里推入一个方法，这个方法也比较简单，如果有传入方法，那么便执行这个方法，如果有传入第二个参数则作为方法的执行环境调用。如果没有传入方法，那么便判断`_resolve`是否存在，这里是使用了闭包来储存`_resolve`，因为`_resolve`的值是在下面进行判断的，如果没有传入方法以及有Promise的话，则让`_resolve`作为`resolve`方法。所以如果`_resolve`存在的话则调用`_resolve`，则意味着这个Promise进入`resolve`状态，那么设置的`then`方法则可以触发了。  

然后便是判断是否处于等待状态，如果没有处于等待状态则执行`timerFunc`，从而去清空任务队列。而平时常用`this.$nextTick`则是定义在另一个位置，代码如下。

```javascript
Vue.prototype.$nextTick = function (fn: Function) {
	return nextTick(fn, this)
}
```
可以看到只是默认帮我们把执行上下文绑定了。

## 总结
可以从不复杂的nextTick代码实现上看到很多优雅的写法，像函数柯里化和闭包感觉在作者手里信手拈来，活灵活现。我们还可以从中看到学到一些平时不常见的知识点，也可以看到在一个框架中实现一个完整的功能需要考虑哪些方面。