---
title: 浏览器事件
date: 2020-06-30 18:50:00
tags:
- Event
---
<!-- more -->
### 事件绑定和解绑
#### DOM0的事件绑定和解绑
浏览器的`dom`进行了多次的升级，第一版本：`dom0`
```javascript
const el = document.querySelector('.box');
el.onclick = handleClick; // 绑定事件
function handleClick (e) {
  console.log('click', e);
}
el.onclick = null; // 卸载事件
```
第一版的事件绑定方式简单粗暴，兼容性也及其的好，但是有几个弊端：

1.对于一种事件，只能绑定一个监听函数，如下：
```javascript
const el = document.querySelector('.box');
el.onclick = handleClick;
el.onclick = handleClick1;
function handleClick (e) {
  console.log('click', e);
}
function handleClick1 (e) {
  console.log('click1', e);
}
// 点击将会打印 click1
```

2.无法指定事件处理函数的时期或阶段，`dom0`中只有冒泡，没有捕获。
#### DOM2级事件的绑定和解绑
由于`dom0`级事件的缺陷，就有了`dom2`级事件，`dom2`级事件中使用`el.addEventListener`绑定事件，使用`el.removeListener`卸载事件。具体用法如下：
```javascript
const el = document.querySelector('.box');
el.addEventListener('click', handleClick); // 绑定事件
function handleClick (e) {
  console.log('click', e);
}
el.removeEventListener('click', handleClick); // 卸载事件
```
`dom2`级事件支持给一个事件绑定多个事件监听函数：
```javascript
const el = document.querySelector('.box');
el.addEventListener('click', handleClick);
el.addEventListener('click', handleClick1);
function handleClick (e) {
  console.log('click', e);
}
function handleClick1 (e) {
  console.log('click1', e);
}
// 点击将会依次打印 click click1
```
在`Chrome`中也可以看到事件确实被绑定了两次：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/372445/1592983401485-5bc8ea5c-db02-4b41-b298-6b89d821e6d1.png#align=left&display=inline&height=204&margin=%5Bobject%20Object%5D&name=image.png&originHeight=408&originWidth=768&size=67828&status=done&style=none&width=384)<br />如果通过`dom2`级事件绑定了多个监听函数，卸载的时候，也必须一个一个卸载：
```javascript
const el = document.querySelector('.box');
el.addEventListener('click', handleClick, false);
el.addEventListener('click', handleClick1, false);
function handleClick (e) {
  console.log('click', e);
}
function handleClick1 (e) {
  console.log('click1', e);
}
el.removeEventListener('click', handleClick, false); // 卸载绑定的 handleClick 函数
el.removeEventListener('click', handleClick1, false); // 卸载绑定的 handleClick1 函数
```
#### DOM2级事件指定事件流的方向
`dom2`级事件还可以第三个参数来指定事件流的方向，默认是`false`、表示在冒泡阶段执行绑定的事件处理函数。
```javascript
const wrapper = document.querySelector('.wrapper');
const el = document.querySelector('.box');
el.addEventListener('click', handleClick, false);
wrapper.addEventListener('click', handleWrapperClick, false);
document.addEventListener('click', handleDocumentClick, false);
window.addEventListener('click', handleWindowClick, false);

function handleClick (e) {
  console.log('click', e);
}
function handleWrapperClick (e) {
  console.log('wrapepr click', e);
}
function handleDocumentClick (e) {
  console.log('document click', e);
}
function handleWindowClick (e) {
  console.log('window click', e);
}
// 依次会打印 click、wrapper click、document click、window click
```
相反，也可以设置指定在捕获阶段进行执行绑定的处理函数
```javascript
const wrapper = document.querySelector('.wrapper');
const el = document.querySelector('.box');
el.addEventListener('click', handleClick, true);
wrapper.addEventListener('click', handleWrapperClick, true);
document.addEventListener('click', handleDocumentClick, true);
window.addEventListener('click', handleWindowClick, true);

function handleClick (e) {
  console.log('click', e);
}
function handleWrapperClick (e) {
  console.log('wrapepr click', e);
}
function handleDocumentClick (e) {
  console.log('document click', e);
}
function handleWindowClick (e) {
  console.log('window click', e);
}
// 依次会打印 window click、document click、wrapper click、click
```
最后，附一张`DOM`事件流的图：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/372445/1592985265879-4b51a997-bf33-488f-8ce3-ccad2304cce2.png#align=left&display=inline&height=236&margin=%5Bobject%20Object%5D&name=image.png&originHeight=252&originWidth=367&size=33672&status=done&style=none&width=343)
#### IE中的事件DOM2级事件绑定
`IE`浏览器有一套自己的`DOM2`事件绑定方法，通过`attachEvent`绑定事件，通过`detachment`解绑事件。
```javascript
const el = document.querySelector('.box');
el.attachEvent('click', handleClick); // 绑定事件
el.detachEvent('click', handleClick); // 解绑事件
function handleClick (e) {
  console.log('click', e);
}
```
不同的是，`attachEvent`不支持第三个参数，所以无法指定事件流的方向，默认的只有冒泡。
#### 通过外观模式模式封装统一的事件绑定接口
```javascript
event: {
  on (el, event, fn, useCapture = false) {
    if (el.addEventListener) {
      el.addEventListener(event, fn, useCapture);
    } else if (el.attachEvent) {
      el.attachEvent(event, fn);
    } else {
      el[`on${event}`] = fn;
    }
  },
	off (el, event, fn, useCapture) {
    if (el.removeEventListener) {
      el.removeEventListener(event, fn, useCapture);
    } else if (el.deachEvent) {
      el.deachEvent(event, fn);
    } else {
      el[`on${event}`] = null;
    }
  }
}

const el = document.querySelector('.box');
event.on(el, 'click', handleClick, false); // 绑定事件
event.off(el, 'click', handleClick, false); // 卸载事件
function handleClick (e) {
  console.log('click', e);
}
```
#### 阻止默认事件和停止事件冒泡
**默认事件**指的是事件的默认行为，比如：在网页上，鼠标右击，默认会出现浏览器的默认的菜单功能：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/372445/1593481385639-59f2bf58-a8cd-4296-a6a3-525f43f1f074.png#align=left&display=inline&height=468&margin=%5Bobject%20Object%5D&name=image.png&originHeight=936&originWidth=692&size=201588&status=done&style=none&width=346)<br />但有时候我们需要自定义这个行为，如：在很多的在线文档，需要自定义鼠标右击的行为，来增加更多的功能等等。<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/372445/1593481505563-3bfa7991-d376-4b57-9b72-2f674b4a3095.png#align=left&display=inline&height=342&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=736&size=41115&status=done&style=none&width=368)<br />这个时候就需要阻止掉默认的事件了，阻止默认事件很简单，在事件的处理函数中，直接调用`event`的`preventDefault`方法就可以了。
```javascript
el.addEventListener('click', function(event) {
  event.preventDefault(); // 阻止事件的默认行为
  // ... 处理自己的逻辑
})
```
事件冒泡前面已经说过了，就是事件在触发之后，会一级一级网上传播，一直传递到`window`，如果祖先元素没有刚好绑定了和子元素相同的事件，则祖先元素该事件的处理函数也会执行，想取消冒泡也很简单，调用`event`的`stopPropagation`方法即可。
```javascript
el.addEventListener('click', function(event) {
  event.stopPropagation(); // 阻止事件传播
  // ... 处理自己的逻辑
})
```
需要注意的是，在`IE`下，`event`对象被绑定在`window`下，并且取消事件传播的方式也不一样，如下：
```javascript
el.addEventListener('click', function(event) {
  const e = event || window.event; // 兼容IE
  if (e.stopPropagation) {
    e.stopPropagation();
  } else {
    e.cancelBubble = false; // 阻止事件传播
  }
  // ... 处理自己的逻辑
})
```
#### 事件冒泡和事件捕获的应用场景
大多数情况下，我们只需要事件处理函数在我们绑定的元素触发即可，但事件冒泡和事件捕获也有特殊的用途。
```html
<nav>
  <a href="">link1</a>
  <a href="">link2</a>
  <a href="">link3</a>
  <a href="">link4</a>
  <a href="">link5</a>
  <a href="">link6</a>
</nav>
```
比如有这样的一段`HTML`结构，需要点击不同的链接进行不同的逻辑处理，通常的做法是：

1. 获取`nav`下所有的`a`标签
1. 循环获取到的标签列表，依次给每个元素绑定上事件

这样做的话性能很低，如果利用事件冒泡的特性，直接给`nav`绑定上点击事件即可，当点击`a`标签的时候，由于事件冒泡，`nav`标签也会接受到，通过`event.target`即可判断具体是点击了哪一个标签，再进行逻辑处理即可，这也是**事件冒泡的一种经典应用**。<br />
<br />再看另外一种**事件捕获的应用场景**：如果有个`js`生成的`dom`元素, 里面绑定了一些`click`事件, 我们想在不`hack`原先代码的情况下, 把`dom`里面的事件拦截，就可以使用事件流捕获的特征，在最外层的元素的事件处理函数中直接`event.stopPropagation()`即可。
#### 自定义事件
除了浏览器内置的事件，如：点击、滚动等等，用户还可以自定义事件，如下：
```javascript
// 定义事件，第二个参数是传递给事件的数据，可以在事件的回调函数中通过e.detail来获取到
const myEvent = new CustomEvent('myEventName', {
  detail: {
    name: 'my event data'
  }
});
// 绑定事件
window.addEventListener('myEventName', function (e) {
  console.log(e.detail); // {name: 'my event data'}
});
// 触发事件
window.dispatchEvent(myEvent); // 低版本 ie 浏览器通过 window.fireEvent(myEvent); 来触发事件
```
这也就是设计模式中的**发布订阅模式。**
#### 提高事件的处理效率
有时候，事件频繁触发会导致网页性能低下，有时候还会给服务器造成很大的压力，通常的做法是使用**函数节流**和**函数防抖**的方式提高事件处理的效率，两者区别在于，函数防抖指的是，只有在事件触发之后间隔的时间超过了设定好的阈值，事件处理函数才会真正的执行，而**函数节流**指的是在一定的事件间隔内，不管事件触发多少次，我都当作一次来处理。<br />以下是简单的函数节流和防抖实现：
```javascript
function debounce(fn,delay){ // 函数防抖
  var timer = null
  //  清除上一次延时器
  return function(){
    clearTimeout(timer)
    //  重新设置一个新的延时器
    timer = setTimeout(() => {
      fn.call(this)
    }, delay);
  }
}

function throttle(fn,delay){ // 函数节流
  // 记录上一次函数出发的时间
  var lastTime = 0
  return function(){
    // 记录当前函数触发的时间
    var nowTime = new Date().getTime()
    // 当当前时间减去上一次执行时间大于这个指定间隔时间才让他触发这个函数
    if(nowTime - lastTime > delay){
      // 绑定this指向
      fn.call(this)
      //同步时间
      lastTime = nowTime
    }
  }
}
```
实际生产环境下，如果项目使用了`lodash`，可使用`lodash`内置的**debounce**和**throttle**函数。或者通过**`throttle-debounce`**npm包，他们提供了函数节流和函数防抖更多的高级特性。
#### 【新】addeventListener的第三个参数
addEventListener的第三个参数除了是boolean值之外，还可以是一个对象，可选的参数有：
```javascript
el.addEventListener('click', handler, {
  capture: false, // 事件流的方向是否是捕获的方式进行
  once: false, // 该事件是否只绑定一次
  passive: false // 提高滚动性能，尤其是在移动端
});
```
在`Chrome 83.0.4103.116（正式版本） （64 位）`中已支持，默认都是`false`。在移动端中，`touch`事件的`passive`默认值为`true`，原因是为了提高移动端滚动的性能。<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/372445/1593308874251-c13cd753-d5a9-4a6e-ba64-d4614cfcd72c.png#align=left&display=inline&height=183&margin=%5Bobject%20Object%5D&name=image.png&originHeight=366&originWidth=852&size=69847&status=done&style=none&width=426)<br />在移动端中，当用户滚动屏幕时，会触发`touchstart、touchmove、touchend`一系列的事件，如果用户在`touchstart`事件中阻止了默认事件：
```javascript
window.addEventListener('touchstart', function(e) {
  e.preventDefault();
  console.log('touchstart');
});
```
则页面将不能滚动，然而，当用户触发`touchstart`事件的时候，浏览器并不知道用户是否会在事件的回掉函数中阻止默认事件，所以浏览器只有执行完事件的回掉函数才知道用户是否阻止了默认事件，当事件的回掉中逻辑比较复杂的时候，滚动的性能就非常的差，尤其是在`touchmove`事件中。<br />在`chrome`下控制台会抛出错误：<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/372445/1593309756859-f92d7e9d-7507-4743-9ea1-ef91893b6c0c.png#align=left&display=inline&height=166&margin=%5Bobject%20Object%5D&name=image.png&originHeight=332&originWidth=1716&size=99409&status=done&style=none&width=858)<br />
<br />提示我们不能在`passive`为`true`时，在事件的回掉函数中阻止默认事件，因为`chrome`为了提高移动端的滚动性能，把`passive`的第三个参数默认设置为了`true`，表示在事件的回掉函数中不会阻止默认事件。<br />
<br />
