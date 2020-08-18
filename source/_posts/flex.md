---
title: Flex布局
date: 2020-06-22 16:00:00
tags:
  - js
---
## 前言

布局的传统解决方案，基于盒状模型，依赖 display 属性 + position 属性 + float 属性。

它对于那些特殊布局非常不方便，比如，垂直居中。

2009年，W3C 提出了一种新的方案 —— Flex 布局。

可以简便、完整、响应式地实现各种页面布局。目前，它已经得到了所有浏览器的支持。

Flex是Flexible Box的缩写，意思为弹性布局，

<!-- more -->

## 注意

### 1. 任何⼀个容器都可以指定为 Flex 布局
```js
display: flex;
```
⾏内元素也可以使⽤ Flex 布局
```js
display: inline-flex;
```

### 2. 设为 Flex 布局以后，子元素的 `float / clear / vertical-align`属性将失效
Flex 布局有两个轴，一个是主轴(默认是水平方向)，一个是交叉轴(默认是垂直方向) 

具体可以看下面的图

### 3.Flex 的兼容性问题

在 `webkit` 内核的浏览器上要加上 `-webkit` 的前缀
```js
display: -webkit-flex;
display: flex;
```

## 图解Flex原理

我们来简单了解下flex布局的⼀些属性：

![](https://user-gold-cdn.xitu.io/2020/6/22/172db9b4515beee6?w=1722&h=1056&f=png&s=420154)

如上图所示：

Flex容器中默认存在两根轴：⽔平主轴（X轴）为main axis，垂直⽅向的Y轴为cross axis。

* X轴的开始位置与容器边框的交叉点为main start，结束位置的交叉点为main end。

* Y轴的开始位置与容器边框的交叉点为cross start，结束位置的交叉点为cross start。


* flex容器中的元素默认沿主轴排列，单个元素占据的主轴空间叫main-size，占据的垂直轴空间叫cross-size。

## Flex盒子的属性

下⾯了解⼀下flex盒⼦的⼀些属性：

| # | 属性 | 属性释义 | 属性值 |
|------|------------|------------|-------|
| 1  | flex-direction| 决定主轴的⽅向，即项⽬的排列⽅向row（默认值）：主轴为⽔平⽅向，从左到右排列  | row-reverse：主轴为⽔平⽅向，从右到左排列<br>column：主轴为垂直⽅向，从上到下排列<br>column-reverse：主轴为垂直⽅向，从下到上排列 |
| 2  | flex-wrap | 默认情况下，项⽬都在⼀条轴线上排列，flex-wrap这个属性定义：如果⼀条轴线排列不下，剩余的项⽬应该如何排列。 | 默认情况下，项⽬都在⼀条轴线上排列：<br>flex-wrap这个属性定义：如果⼀条轴线排列不下，剩余的项⽬应该如何排列。<br>nowrap（默认值）：不换⾏<br>wrap：换⾏，第⼀⾏在上⽅<br>wrap-reverse：换⾏，第⼀⾏在下⽅ |
| 3  | flex-flow| 该属性是flex-direction和flex-wrap的组合简写形式，所以默认值是row nowrap | 语法<br>flex-flow：\<flex-direction> \|\| \<flex-wrap>;|
| 4  | justify-content | 项⽬在主轴上的对⻬⽅式 flex-start（默认值）：左对⻬|flex-end：右对⻬<br>center：居中<br>space-between：两端对⻬，项⽬之间的间隔相等<br>space-around：每个项⽬两侧的距离相等，所以项⽬之间的间隔⽐项⽬的边框⼤⼀倍 |
| 5  | align-items  | 项⽬在垂直轴上如何对⻬ | flex-start：垂直轴的起点对⻬<br>flex-end：垂直轴的终点对⻬<br>center：垂直轴的中点对⻬<br>baseline：项⽬的第⼀⾏⽂字的基线对⻬<br>stretch：若项⽬未设置⾼度或者为auto，将占满整个容器的⾼度 |
| 6  | align-content | 定义了多根轴线的对⻬⽅式，如果项⽬只有⼀根轴线，那么该属性不起作⽤。 | flex-start / flex-end / center / space-between / space-around / stretch。 |

 

## Flex盒子中元素的属性

下面这些是设置在flex盒子里的项目的属性：

| # | 属性 | 属性释义 | 属性值 |
|------|------------|------------|-------|
| 1  | order          | 定义项⽬的排列顺序，数值越⼩越靠前排列，默认为0         | 整数值 |
| 2  | flex-grow        | 定义项⽬的放⼤⽐例，默认为0，即如果存在剩余空间也不放⼤ |  数值 <br>说明：如果所有项⽬的flex-grow属性都为1，则他们将等分剩余空间 <br>如果⼀个项⽬的flex-grow属性为2，其他项⽬为1，则前者占据的剩余空间⽐其他项⽬多⼀倍。|
| 3  | flex-shrink       | 定义项⽬的缩⼩⽐例，默认为1，即如果空间不⾜，该项⽬将缩⼩ | 数值<br>说明：如果所有项⽬的flex-shrink属性都为1，当空间不⾜时，都将等⽐缩⼩<br>如果⼀个项⽬的flex-shrink属性为0，其他项⽬都为1，则空间不⾜时，前者不缩⼩。|
| 4  | flex-basis   | 定义在分配多余空间之前，项⽬占据的主轴空间的⼤⼩（默认auto）| 可设置为具体px / auto，如果设置的是具体的px，则项⽬将占据固定的空间（详单于设置了⼀个固定的width或者height） |
| 5  | flex  | 是flex-grow / flex-shrink / flex￾basis的简写，默认为0 1 auto，后两个属性可选| 该属性有2个快捷值 auto（1 1 auto） 和 none （0 0 auto）建议优先使⽤这两个属性，⽽不是单独写3个分离的属性，因为浏览器会推算相关值 |
| 6  | align-self | 属性允许单个项⽬有与其他项⽬不⼀样的对⻬⽅式，可覆盖align￾items属性，默认值为auto，表示继承align-items的属性，如果没有⽗元素，则等同于stretch。| 可选值与align-items相同，以及多加⼀个auto。 |


## 应用

### 1. 设置垂直水平居中

```HTML
<div class="outer">
  <div class="inner">内部盒子</div>
</div>
```

```CSS
.outer {
    display: flex;
    justify-content: center;
    align-items: center
}
```


### 2. 设置左边栏固定，右边栏自适应布局(或类似布局)
```HTML
<div class="outer">
  <div class="left">左边盒子</div>
  <div class="right">右边盒子</div>
</div>
```
```CSS
div {
  border-radius: 10px;
  color: #fff;
  font-weight: bold;
  text-align: center;
}
.outer {
  height: 300px;
  display: flex;
  padding: 10px;
  background: #e8e8e8;
}
.left {
  flex: 0 0 100px;
  background: #000;
}
.right {
  flex: auto;
  margin-left: 10px;
  background: #1487B4;
}
```

### 3. 设置浮动自适应布局
```HTML
<div class="panel">
  <div class="outer">
    <div class="inner" style=""></div>
    <div class="inner" style=""></div>
    <div class="inner" style=""></div>
    <div class="inner" style=""></div>
    <div class="inner" style=""></div>
    <div class="inner" style=""></div>
    <div class="inner" style=""></div>
    <div class="inner" style=""></div>
    <div class="inner" style=""></div>
  </div>
</div>
```
```CSS
.outer {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
.inner {
  height: 50px;
  width: 100px;
  margin: 10px 0;
  line-height: 100px;
  background: #1487B4;
}
```

可参考链接：

[link🔗](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

