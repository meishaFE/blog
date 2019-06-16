---
title: js动态修改@keyframes关键帧
date: 2019-06-16 16:00:00
tags:
  - js
---

## 前言

最近做海底冒险游戏时遇到了一个小需求，就是一个海底背景的动画效果要随着用户点击加速减速按钮的交互去实时滚动背景:
<!-- more -->
![avatar](../images/miner.gif)

这里首先想到的是用keyframes去实现，因为可以实时获取到图片当前的位置，所以就通过改变keyframes中关键帧的top值来实现下降和上升的效果：
```javascript
      @keyframes 下降 {
        0% {
          top: (背景图当前位置)px
        }
        100% {
          top: 背景图末尾位置
        }
      }
      或者
    @keyframes 上升 {
        0% {
          top: (背景图当前位置)px
        }
        100% {
          top: 背景图开头位置
        }
      }
```
大致想法就是这样，具体实现是怎样的呢？ 请接着往下看


## 实现

首先插入新的标签：
```javascript
  mounted() {
    if (!dynamicStyles) {
      dynamicStyles = document.createElement('style');
      dynamicStyles.type = 'text/css';
      document.head.appendChild(dynamicStyles);
    }
  }
```
然后定义一个动画：
```javascript
    backgroundAnimation(name, from, to) { 
      dynamicStyles.sheet.insertRule(
      `@keyframes ${name} {0% { top: ${from}rem; }100% { top: ${to}rem; }`,
      dynamicStyles.length
    );
  }

```
接下来在需要点击按钮时调用动画改变的函数就可以了：
```javascript
 this.backgroundAnimation('up', -52.67, -1);
 this.$refs.sea.style.animation = `up ${this.totalTime}s linear`;
```
后面发现这样虽然可以实现效果，但是因为计算当前位置本身就有一定的误差，导致图片有时不是在当前的位置变换上升下降的状态,所以改为了用translate：
```javascript
this.$refs.sea.style.webkitTransform = `translate(0, ${top}rem)`;
```
就简单的搞定了。。。（没直接改变top值是因为dom的top值修改会触发重绘，页面会一卡一卡的）
写到这里就算over啦~ 虽然没用到上面的方法但是这种方法应该也会适用于一些特定的场景，也算涨了姿势，谢谢大家~