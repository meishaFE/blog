---
title: element-ui源码解析一（button篇）
date: 2019-03-01 13:45:46
tags:
- element-ui 
- button
- 组件
- 源码
---

## 前言
转眼间已经工作一年了，在这期间主要熟悉了点业务代码，🤔em。。。是时候提升一下自己的格调了（看点源码）。但一开始又不好看太深（Vue 源码），索性就先学习 Element 里面的组件是怎么写的吧，毕竟这个看起来会比 Vue 好理解点，对工作也能产生直接的帮助，希望事实也是这样🙃。所以 Element 又该怎么看呢，就挑一个组件慢慢看罗（最后再学习其目录架构罗）。本篇文章就是从最简单实用的 button 组件下手啦。废话了这么多，我得赶紧切入主题啦。

<!-- more -->
Element源码地址：https://github.com/ElemeFE/element
Element文档：http://element-cn.eleme.io/#/zh-CN/component/button

## 怎么看
这里先简要说下我看组件源码的方法，仅供大家参考，不喜勿喷🙄。
首先 Element 有几个版本，我看的是基于 Vue 的版本，所以每个组件到底就是一个 vue 文件，就和我们平时工作写的代码一样，写好一个 vue 组件，然后在需要的页面引入即可。不过重要的是如何写好这个组件（健壮吗，可扩展吗，易维护吗等）。一个 vue 组件一般可分为三部分，`template`、`script` 和 `style`。在这里我们就不考虑 `style` 了，直接在页面引用 Element 的样式就好，这不是我们主要关心的，我们只要知道 Element 的样式一般是这样（`el-组件名--状态`，比如 `el-button--primary`）命名的就行。所以我们组件里是没写 `style` 部分的，这样做能帮我们省下好多时间和精力。
```javascript
// 直接在页面中引入 Element 的样式
<link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
```

## template 部分
那么接下来我们就先看看 `template` 的部分怎么写。其实这部分是很简单的（对于这个组件来说😁），我们可以打开 Element 文档看一下 button 的外观样式，再来写这部分。
假设你已经看过 button 组件的大部分外观后，我们就可以在脑海中先想一下（抽离并化简一下 `html` 结构的公共部分），大概就是一个 `div`（button 标签）里面包了一个 `i` 图标和 `span` 文本这样的结构，嗯好像是这样，那就试着写一下吧！（提示：一般最外层的样式都是用 `el-组件名` 包起来）
```javascript
<template>
  <button class="el-button">
    <i></i>
    <span></span>
  </button>
</template>
```
看上面的结构像那么回事，也简单明了。不过，然后呢😯。。。
然后就是我们的 `script` 部分啦，这个才是组件的灵魂所在，重中之重，也是需要我们去啃的部分。好在这个组件简单，让我们继续往下看吧。

## script 部分
我们看这部分的时候，可能无从下手，但其实还是有点门道的。敲锣啦👏👏👏。。。。不管神马组件，都有三个较为重要的组成部分：`props`、`event` 和 `slot`，这三个部分是组件对内对外沟通的桥梁，使得组件变得灵活起来。所以这三个 api 在发布之前一定构思好和确定好，因为后期再改就很难了，可能就是会牵一发动全身，但后期对组件的处理不应该是这样的效果，而应该是不影响和改动之前的 api，但又可以扩展和新增功能。好的，就让我们一个一个娓娓道来吧👇。
首先看下 `props` 的部分，你需要在脑海中想象一下 button 组件的哪些内容是可变的（根据需要外部传参的改变而改变），不用着急往下看，先好好想一下💤。。。。
...
1）最明显的就是 button 的背景色吧，这显然是可变的，就是 type；
2）然后是有没有图标，就是 icon；
3）还有就是有没有禁用，就是 disabled；
4）再来是有没有圆角，就是 round；
5）尺寸大小也是可变的吧，就是 size；
6）好像按钮还可以是文本的样子，就是 plain;
....
好了，那我们就试着写一下 `props` 部分吧！（注意：`props` 的部分最好用对象的写法，这样能够对每个属性进行自定义设置，相比数组的写法，也更为规范严谨）
```javascript
<script>
export default {
  props: {
    type: {
      type: String,
      default: ''
    },
    size: {
      type: String,
      default: 'medium'
    },
    icon: {
      type: String,
      default: ''
    },
    disabled: Boolean,
    plain: Boolean,
    round: Boolean
  }
}
</script>
```
接下来是 `slot` 部分啦，如果不懂 `slot` 用法的同学可以先出门左拐学习一下再来✋。很明显，对于 button 组件来说，文本就是 `slot` 啦，所以 `template` 里面的内容可以小改一下，代码如下：
```javascript
<template>
  <button class="el-button">
    <i></i>
    <span><slot></slot></span>
  </button>
</template>
```
然后是 `event` 部分，很显然啦，按钮能有什么功能呢，就是点击嘛，没了，所以它也就一个事件，就是当按钮被点击的时候，我们需要触发一个事件向上传递，也就是 `$emit`。于是乎，我们把事件添加到组件中，代码如下：
```javascript
<template>
  <button
    class="el-button"
    @click="handleClick">
    <i></i>
    <span><slot></slot></span>
  </button>
</template>
<script>
export default {
  props: {
    ...
  },
  methods: {
    handleClick (e) {
      this.$emit('click', e);
    }
  }
}
</script>
```
好像 `event` 的部分就那么多，嗯，是的，比想象中的简单✊。。。。

## 再看 template 部分
你以为组件写完了，不，并没有，你不觉得 `template` 里面太空了么，而且 `props` 这部分的属性都还没用上呢（只是声明了一下），所以我们还需要完善点东西。。。
比如 `slot` 部分吧，`$slots.default` 可以获取 `slot` 中的内容，不过这里需要加个判断，因为用户可能没有传文字，那我们就不用渲染了；
又比如图标 `i` 的部分，和 `slot` 一样，有传值我们才渲染，所以也加个判断（这里 icon 的值为 `el-icon-图标名` 格式）；
```javascript
<template>
  <button
    class="el-button"
    @click="handleClick">
    <i :class="icon" v-if="icon"></i>
    <span v-if="$slots.default"><slot></slot></span>
  </button>
</template>
```
再看 `props` 中的属性，其实大部分是用来控制样式变化的，比如 type，size，round，disabled，plain。。。所以就为组件加上 `class` 吧。
```javascript
<template>
  <button
    class="el-button"
    @click="handleClick"
    :disabled="disabled"
    :class="[
      type ? 'el-button--' + type : '',
      size ? 'el-button--' + size : '',
      {
        'is-disabled': disabled,
        'is-plain': plain,
        'is-round': round
      }
    ]">
    <i :class="icon" v-if="icon"></i>
    <span v-if="$slots.default"><slot></slot></span>
  </button>
</template>
```
至此我们就写完了一个较为完整的 button 组件，是不是给人一种（这么简单的么）的感觉，虽然它还不够完善，但也覆盖了源码 90% 的部分，剩下的 10% 大家可以自己去补充补充。其实组件主要看你🤔思考🤔得有多全面，想的越多写的越多。
## 结语
顺着上面的思路，我们就可以有个较为明了的方法去看其他组件的源码，应该会轻松一些吧😂。当然这里仅仅只是起一个抛砖引玉的作用，主要是因为它确实简单还实用（很适合敲门砖的角色），后面看到其它复杂的组件也许就不会这样想了😝。
总而言之，革命尚未成功，同志仍需努力。撸起袖子加油干💪！！！