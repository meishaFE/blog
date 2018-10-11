---
title: Fabricjs学习笔记
date: 2018-10-11 09:22:15
tags:
- canvas
---

## Fabricjs学习笔记

 	此次的项目是一个物理类游戏，同时涉及到的交互也比较多，原生的canvas画布可能满足不了项目的需求，因为原生canvas不提供点击画布内元素产生各类交互的功能，故去寻找了一个符合项目需求的canvas库--Fabricjs，这个库是在canvas上面封装了一层自己的逻辑，为canvas元素提供了交互式对象，使用Fabricjs可以把自己画的每一个道具或者图形当成一个对象，然后通过去修改对象属性去实现移动旋转或是变形。下面是项目时常用到的代码段：

```javascript
// 画矩形
var canvas = new fabric.Canvas('canvas')
var rect = new fabric.Rect({
    left: 100,
    top: 100,
    fill:'red',
    width: 20,
  	height: 20
})
canvas.add(rect)
// 画圆
var circle = new fabric.Circle({
    radius: 20,
    fill:'green',
    left: 100,
    top: 10
})
canvas.add(circle);
// 画三角形
var triangle = new fabric.Triangle({
    width: 20,
    height: 20,
    fill: 'red',
    left: 50,
    top: 50
})
canvas.add(triangle);
// 画线(要知道两点坐标)
var line = new fabric.Line([x1,y1,x2,y2], {
    stroke: '#000'
})
canvas.add(line);
// 画椭圆
var ellipse = new fabric.Ellipse({
  rx: 45,
  ry: 80,
  fill: 'yellow',
  stroke: 'red',
  strokeWidth: 3,
  angle: 30,
  left: 100,
  top: 100
});
canvas.add(ellipse);
// 画图片
var img1 = this.$refs.img1
var image = new fabric.Image(img1, {
    left 10,
    top: 10,
    angle: 30 //旋转角度
    opacity： 0.85 // 透明度
})
canvas.add(image)
```

​	这里面的除了画线要两点坐标外其他的都是left和top来控制图形的位置， 默认是左上角那个点，如果想更改为中点的话在对象里面加originX: 'center',originY: 'center',就可以让图形的坐标点定为图形中点。

如果想要监听并操作这些图形，用Fabric自带的监听事件即可：

```js
// 鼠标按下
this.canvas.on('mouse:down',(event)=>{
//如果点击到了canvas上面的图形，则event.target为这个图形对象，可以任意做属性的修改，另外可以用event.pointer.x和event.pointer.y来取到鼠标当前坐标用来做边界判断或是其他操作。
})
//鼠标移动
this.canvas.on('mouse:move',(event)=>{
})
// 鼠标移入&移出(两个加起来实现hover效果)
this.canvas.on('mouse:over',(event)=>{
})
this.canvas.on('mouse:out',(event)=>{
})
// 对象拖动
this.canvas.on('object:moving',(event)=>{
})
// 对象旋转
this.canvas.on('object:rotating', (event) => {
})
// 对象增加
this.canvas.on('object:added', (event)=>{
// 可以自己保存一份然后执行撤销回退时用
})
```

​	选中物体后Fabricjs自带了一个控制框，如下图：

![image-20181010171117028](/var/folders/8c/8_3m9d954_b7644sj2t6xbrh0000gn/T/abnerworks.Typora/image-20181010171117028.png)



使用下面这个代码可以让控制框的小方块变成有填充色的,改变填充色使用cornerColor: '#ff0000'

fabric.Object.prototype.set({
    transparentCorners: false
});

如果想去掉连接的红色线可以用 object.hasBorders = false

如果想去掉绿色的控制柄可以用 object.hasControls =  false

如果想设置点击元素时元素不能被操作 可以用object.selectable = false，但是这个只能隐藏控制框但是元素所在的范围还是比实际上大的（参考上图圆的四个空白区），要让这个范围变成元素范围要再加上 

object.evented = false，让对象不作为事件源。

如果想锁定控制框的某个功能可以使用 lockMovementX、lockMovementY、lockRotation、lockScalingX 、lockScalingY这些属性。



fabricjs的层级问题 ：做需求的时候发现点击两点后没正确连到想要的位置，图为想要的效果：

![image-20181010175807861](/var/folders/8c/8_3m9d954_b7644sj2t6xbrh0000gn/T/abnerworks.Typora/image-20181010175807861.png)



多次试验后发现是作图时层级有问题，因为是先画两个点再画直线，所以线的层级比圆点高,导致点圆点时点到了线上,才连错了位置, 解决方法是把线的层级调低,画到圆点底下

代码为

​        ```line.moveTo(0);```

这样使用不好的点是会让line在canvas对象数组中排到第一位，就会影响到撤销回退时导致线没被撤销，解决方法是自己保存一份对象数组用来做撤销，或者是不要moveTo(0)，moveTo小圆点index的后一个位置。

设置背景图：

```js
this.canvas.setBackgroundImage('/game/Group3@2x.png',this.canvas.renderAll.bind(this.canvas))
```

设置浮动到对象上面后的鼠标样式

```js
object.hoverCursor = 'help';
```

设置鼠标在画布上面的样式

```js
this.canvas.defaultCursor = `url('/game/static/stage/${this.cursorToolImgs[idx]}') 10 20, auto`
// 第一个参数是设置鼠标为一个图片，第二三个是鼠标点击位置，第四个是如果第一个参数无效的话可以有个备选的样式，一般用auto
```

持续更新。。。