---
title: 关于setInterval切换页面时不工作
date: 2018-05-04 11:48:46
tags:
- js
- setInterval
- 计时器
---

## 前言

这是在做追逐小游戏时遇到的一个问题，其实就是个简易的计时器效果（00:00 ~ 30:00），理所当然的用`setInterval(() => {}, 10)`写了。写完之后发现，在切换页面或者窗口的时候，计时器不会继续计时，直到切换回该页面的时候才会继续进行。同时页面上还有 css 动画，在切走页面时动画仍会继续进行，于是`setInterval`计时器就和 css 动画就不同步了。

<!-- more -->

最初的想法就是把`css`动画在切换页面的时候暂停一下，等到切换回该页面的时候继续播放动画，也就是通过监听`onblur`、`visibilitychange`或`minimize`等事件，把`animation-play-state`这个属性设置为`paused`或者`running`，代码如下：

```javascript
// 当窗口失去焦点和获得焦点
window.onblur = function() {};
window.onfocus = function() {};

// 当用户切换页面或者最小化窗口时，document.hidden的值会变为true
document.addEventListener(‘visibilitychange’, function(){
  if (document.hidden) {
    // todo
  } else {
    // todo
  }
});
```
不过在尝试了之后感觉不是很稳定，就是以不同方式切走该页面有时会无效。而且产品的要求是最好在切走页面的时候也能够继续计时。所以就开始各种百度啦！！!得到的结果是：喔，原来是浏览器自身的特性，可能是出于性能考虑吧，如果前窗口处于非激活状态时，`setInterval`就会停止运行或者以较慢的速度运行，当然这不是绝对的，还和所设置的时间间隔有关，如果时间间隔较长（比如2000ms），切换页面时`setInterval`还是会继续进行的，如果时间间隔较短（如几十毫秒），`setInterval`就会停止工作，具体哪个时间节点我还不是很清楚。而我这个计时器时间间隔是10ms，所以切换页面浏览器是不工作的。

当时就一股脑的想着怎样让`setInterval`一直工作呢，忽然想起了 H5 有个 web Worker 这个东西，它可以开启一个子线程，该线程能够独立于主线程工作，并进行通信。也就是说我们可以把计时的操作放在子线程中然后向页面回传数据，而主线程只要负责接收数据并进行展示不就可以了么，于是就尝试了一下，嗯，果然可以，在 Safari、火狐和 Chrome 下切走页面的时候`setInterval`依然能够运行，所以这个问题就这样解决了，代码如下：

```javascript
// 这是主线程
var worker = new Worker("worker.js");  // 创建一个Worker对象
worker.postMessage('hello');  // 向worker发送数据
  worker.onmessage = function(e){  // 接收worker传过来的数据
    console.log(e.data);
  }
```
```javascript
// 这是子线程worker.js
onmessage = function (e){
  setInterval(() => {
    // todo
    postMessage(data);  // 向主线程中发送数据
  }, 10)
}
```

嗯，一切好像都挺合理。。。直到后来，现场演示的时候出现了bug，计时器一直在00:00的状态，没有变化，这个就很尴尬了，想必应该是`new Worker()`没有工作吧，可是为什么不工作呢，噢，原来是演示的时候用了 IE 浏览器，应该是它不支持`new Worker()`，所以得另辟蹊径了。。。后来的解决方案是通过`new Date()`来配合计时，这样切换回页面的时候，由于当前时间是会变化的，我们就可以用现在的时间`new Date()`减去起始的时间 begin 得到的时间差 time 来计时，嗯，应该是个不错的方案了，至少目前没有什么太大问题，具体代码如下：

```javascript
let begin = +new Date();  // 设置起始时间作为参照
let timer = setInterval(() => {
  let time = +new Date() - begin; // 时间差也就是毫秒数
  console.log(time);
}, 10)
```

## 后序
- 可用`setTimeout`来模拟`setInterval`的效果，好处在于`setTimeout`能避免`setInterval`产生的叠加效果，能够更为准确的控制时间间隔，代码如下：

```javascript
function fn() {
  // todo
  setTimeout(fn, 1000);
}
setTimeout(fn, 1000);
```
- 要养成清除计时器的习惯，避免不必要的性能损耗，`setInterval`和`setTimeout`每次执行其实会返回一个 id 值（这个 id 就是个数字），然后我们可以通过这个 id 标识来清除计时器，代码如下：

```javascript
let timeId = setInterval(function() {
  // todo
}, 1000)
clearInterval(timeId);
```
当然我们可能会遇到这样的一种情况：当页面`setInterval`用的较多时，我们在离开或者跳转页面前若是要清除所有的计时器要怎么办？这时我们可以对`setInterval`生成的所有 timeId 进行统一管理，简单来说就是把每个 timeId 都放入一个数组中，页面离开或跳转时再遍历进行`clearInterval`，`setTimeout`也是一样的道理，具体代码如下：

```javascript
let timeArr = [];
let timeId1 = setInterval(function() {
  // todo
  timeArr.push(timeId1);
}, 1000)
let timeId2 = setInterval(function() {
  // todo
  timeArr.push(timeId2);
}, 2000)
// 页面跳转或离开前
timeArr.forEach(function(item) {
　clearInterval(item);
});
```