---
title: Input限制输入之数字篇
date: 2019-03-31
tags:
- Input Component
- JavaScript
- Vue & React
---

阳春三月本是一个踏青的好季节，奈何我是一个热爱工作的“痴情程序员”，当然是选择无止境的加班工作啦。回归正题，这次给大家带来的是最最基础的但是困扰了我好久的限制输入(只能输入数字)功能。之前一直想做篇总结，苦于没有好的“心情”去收拾，这一次借着刚刚在项目中实践过的热情给大家献上一篇基础博文。

<!-- more --->

## 一、背景

输入框作为人机交互最直接最常用最简单的载体，它承载着巨大的重担。Web网页从远古时期的只能“远观”到Web2.0的“亵玩”，用户输出自己想要的内容，输入框成了海量数据来源的重大途径。那么如何让用户舒适地输出自己的想法成了我们这些离用户最近的前端工程师的“首要”任务。当然在用户舒适的同时，我们也要确保从用户那里拿到的数据是我们想要的，不是那些花里胡哨乱七八糟的“脏数据”，避免后续复杂的校验工作。所以，我们需要把这种行为“扼杀在摇篮里”。
一个极其常见简单的例子，我们需要收集用户的手机号。众所周知手机号是纯数字的，但是用户的键盘是含有英文字符，制表符等其他非数字字符的。为了确保数据的可用性，我们需要限制用户的输入行为——只让他们输入数字。这样的解决方案数不胜数。

### 1. 使用`type="number"`属性

强大的`HTML5`集成很多好用的属性与方法，原生`input`标签就拥有`type="number"`这个属性，来限制用户只能输入数字。虽然这种方案已经足够满足移动端H5的需求(自动调起手机的数字键盘)，但是它在PC端就没那么完美了。它在`hover`状态下显示上下两个小箭头影响美观，并且对于小数点，负号没有做个数上的限制，用户依旧能输入非法值。因此这个方案显然不满足我们的野心。

### 2. 提交表单或输入框失焦时校验——显示错误提示并标红输入框

这是目前比较多的实现方案，在表单提交的时候做校验，禁止非法值流入到后端逻辑中去，并给了用户明显友好的提示。但是作为文明讲究效率的现代人，用户需要的是“有错误，给我一时间指出来，我立马就改，或者你直接帮我改好”。所以摒着这个宗旨，来到了我们今天目前我觉得比较好的“终极解决方案”。

### 3. 正则匹配阻断赋值渲染(React) & 强行赋合法值(Vue)

看到标题先不要懵，我们的核心是“正则匹配”。利用强大的正则公式去匹配我们想要得到的数据格式，后面两种方式是针对不同框架的实现而已。因为两种框架之间还是有着一点的区别，其中会涉及到`React`中**受控组件**与**非受控组件**，这一点我会在下文简单阐述一下，若有兴趣可以去[官网](https://react.docschina.org/docs/forms.html)看看官方的解释。

## 二、具体实现

回到重点，在这里我给大家讲述一下，我是如何实现这一方案的。

### 1. 写出合适的正则表达式

匹配数字的正则极其简单，无非就是数字开头数字结尾，中间出现的都是数字——`/^(\d)+/`，如果不允许零开头——`/^([1-9])+/`等等。这一次我就举例这次项目开发中的实际需求——两位小数点的非负数。这也就意味着0可以开头，允许数字后面跟一个小数点并且小数点后面只能有两位数字。直接上表达式——`/^[0-9]+(\.[0-9]{0,2})?(?![0-9])/`。这我相信大家能够看懂。此外进行字符串匹配有两种方法：要么采用字符串的`match`方法，要么采用正则表达式的`test`方法。前者若匹配成功则会返回一个类数组，而后者仅会返回一个布尔值。这里我习惯性的采用了前者。

### 2. `Vue`中实践

代码如下：

```javascript
// template
<template>
  <section>
    <input
      type="text"
      v-model="textVal"
      @input="e => handleChange(e.target.value)"
      placeholder="请输入"
      class="myInputText"
    >
  </section>
</template>

// script
<script>
export default {
  name: "InputTest",
  data() {
    return {
      textVal: "",
      oldVal: ""
    };
  },
  methods: {
    handleChange(val) {
      let value = val;
      if (value) {
        let reg = /^[0-9]+(\.[0-9]{0,2})?(?![0-9])/;
        let result = value.match(reg);
        if (result && result[0] === result["input"]) {
          // this.textVal = value; 自行体会
          this.oldVal = value;
        } else {
          this.textVal = this.oldVal;
        }
      } else {
        this.oldVal = "";
      }
    }
  }
};
</script>
```

是不是很简单？监听输入框的输入事件，对每一次输入做正则匹配处理将输入的值“格式化”成我们想到的“模板数据”。也许很多同行都会纳闷，为何我要在正则不匹配的情况把`oldVal`的值赋给`textVal`。原因是在`Vue`中`input`是不可控的，偷用一个概念来说就是“非受控组件”。也就是说，尽管在不匹配的情况下我们没有写很明确的代码去赋`textVal`的值，但是它仍然变成了我们不想要的样子(输入的值)。所以，我们需要在匹配的情况下保存当次的值为下一次不匹配的情况提供“回退值”。这样就会限制成我们想要的样子。

### 3.`React`中实践

代码如下：

```javascript
import React, { Component } from 'react'

class CommentInput extends Component {
  constructor (props) {
    super(props)
    this.state = {
      textVal: ''
    }
  }

  handleChange = event => {
    let value = event.target.value
    if (value) {
      let reg = /^[0-9]+(\.[0-9]{0,2})?(?![0-9])/
      let result = value.match(reg)
      if (result && (result[0] === result['input'])) {
        this.setState({ textVal: value })
      }
    } else {
      this.setState({ textVal: '' })
    }
  }

  render () {
    const states = this.state
    return (
      <div>
        <input type='text'
          onChange={this.handleChange()}
          placeholder='请输入'
          value={states.textVal}
        />
        {/* 非受控组件
          <input type='text'
          onChange={this.handleChange()}
          placeholder='请输入'
        />*/}
      </div>
    )
  }
}
export default CommentInput
```

相比之下，在`React`中做这样的限制就简单的多，而且呈现出来的效果也比在`Vue`中要好，感兴趣的可以拷贝代码自己跑一下尝试尝试。这得益于啥？没错，我就是我之前提到的“受控组件”。在代码中我也注释出来了，“非受控组件”和“受控组件”最简单明了的区别——是否绑定了内部`state`变量。简单来说，“受控组件”就是组件的状态更新受`React`的控制，只能使用`setState()`进行更新。这样一来我们就可以在限制输入上为所欲为了，也省掉了去定义一个临时不相关变量的步骤。更重要的是，`React`官方推荐在表单中使用"受控组件"，所以许多第三方UI库均保留了这个特性，就算我们使用其他三方封装好的高级组件也能像处理原生组件的方式来处理。而不像`Vue`中当使用`Element UI`的`Input`组件时还需要  `this.$refs.InputRefName.currentValue = value;`这样的方式去强制更新状态。

## 三、总结

个人理解能力与研究深度有限，故文章中有任何不妥，欢迎指出！
