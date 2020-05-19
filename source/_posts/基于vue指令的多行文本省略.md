---
title: 基于vue自定义指令的多行文本省略实现
date: 2020-05-19 10:00:00
tags:
- vue
- directive
---

## 背景

之前在做九宫格平台的时候经常会遇到需要实现多行文本省略的场景，考虑到如果是每个地方都去写调用文本省略的方法的话，代码会显得比较繁琐。于是写了一个自定义指令 `v-clamp`， 这样只用在需要加文本多行省略的元素上加这个指令，就可以实现多行省略，开发效率提高了不少。

<!-- more -->

## 确定需求

首先介绍一下这个指令是怎么使用的

```html
<div>
  <span v-clamp="{
    content: detail.title,
    row: n
  }"
  ></span>
</div>
```

如以上代码所示，目的是要对 `span` 中的内容 `detail.title` 实现两行省略，首先要对 `span` 绑定自定义指令 `v-clamp`， 并且传入一个配置对象， 如 `content`（绑定变量）， `row`（表示要实现 `n` 行省略）。 因此，我们的基本需求就定下来了，接下来是功能的实现。

## 功能实现

### 实现原理

这里的实现主要是通过判断元素中内容的高度是否超过了目标高度来判断是否要省略的。所以这里需要有相应的逻辑处理，如：

1. 如何判断元素内容是否超过目标高度
2. 如何计算元素内容应该在文本的哪个位置进行省略

对于第一个目标，很容易想到先去计算这个元素中内容的一行的高度是多少，然后再去乘相应的行数 `n`， 这样就能算出这个元素的目标高度，然后一比较就知道是否应该进行省略处理。

对于第二个目标，采用的是通过二分法去对目标元素的文本进行裁剪的方法，如果目标元素的高度大于我们所要的目标高度，则进一步采取二分法裁剪，直到元素的高度小于等于目标高度为止。

由于实现过程特殊情况较多，代码较为复杂，以下只展示部分核心代码：

```js

function formatText (node, binding) {

  // 计算目标元素的目标高度
  let maxHeight = calMaxHeight(node, binding);
  // 所绑定变量的值
  let rawContent = binding.value.content;
  // 对目标元素的内容进行处理
  measureText(textContent);
  
  let measuredObj = measureText(textContent);
  let content = measuredObj.subContent;
  let isMultiple = measuredObj.isMultiple;

  node.elm.innerHTML = isMultiple ? `${content.slice(0, -1)}${END_CHAR}` : content;

  /**
   * 计算目标元素的目标高度
   * @param {VNode} node
   * @param {Object} binding
   * @returns {number}
  */
  function calMaxHeight (node, binding) {
    // 获取绑定的变量的值
    let rawContent = binding.value.content;
    // 获取最大行数
    let limitRows = binding.value.row || 1;
    let targetNode = node.elm; // Node that was bound with directive clamp.
    // forceElemWordbreak 用于强制给目标元素加上换行属性
    forceElemWordbreak(targetNode);
    // 通过设置较少的内容取实现目标元素一行的高度的计算，元素宽度较小的场景不适合
    targetNode.innerHTML = '加载中...';
    // computedEffectHeightCSS 用于计算targetNode除内容以外多出来的高度tHeight，如padding/border等
    let tHeight = computedEffectHeightCSS(targetNode);
    // 获取目标元素内容一行的高度
    const targetContentLineHeight = targetNode.offsetHeight - tOtherHeight;
    let maxHeight = targetContentLineHeight * limitRows + tOtherHeight;
  }


  /**
   * 使用二分法内容进行处理。如果内容超过指定行数则进行裁剪
   * @param {string} textStr
   * @param {number} startLoc
   * @param {number} endLoc
   * @param {number} lastSuccessLoc
   * @returns {Object} 返回对象，是否省略与处理后的内容
   */
  function measureText (textStr, startLoc = 0, endLoc = textStr.length, lastSuccessLoc = 0) {
    let midLoc = Math.floor((startLoc + endLoc) / 2);
    let currentStr = textStr.slice(0, midLoc) + END_CHAR;
    if (startLoc >= endLoc - 1) {
      while (targetNode.offsetHeight > maxHeight) {
        targetNode.innerHTML = textStr.slice(0, --endLoc) + END_CHAR;
      }
      return {
        isMultiple: true,
        subContent: textStr.slice(0, endLoc)
      };
    }
    targetNode.innerHTML = currentStr;
    if (targetNode.offsetHeight <= maxHeight) {
      return measureText(textStr, midLoc, endLoc, midLoc);
    }
    return measureText(textStr, startLoc, midLoc, lastSuccessLoc);
  };
}

// 导出指令
export default {
  name: 'clamp',
  inserted: run,
  componentUpdated: (el, binding, node) => {
    if (binding.oldValue.content !== binding.value.content) {
      run(el, binding, node);
    }
  }
};
```

## 一些常见的问题

1. 如要对一个 `div` 下的其中一个 `span` 进行省略判断，可以给该 `div` 绑定一个 `class` 属性，如：

  ```html
  <div v-clamp="{
    content: detail.title,
    row: 1,
    class: targetClass
  }">
    <span>span1</span>
    <span :class="targetClass"></span>
  </div>
  ```

2. 如果目标元素的大小的变化存在过渡动画，可以通过加 `delay` 属性来延迟计算（单位毫秒），如：

```html
<div v-clamp="{
  content: detail.title,
  row: 1,
  delay: 1000
}">
</div>
```
 
3. 对于需要实现省略后显示 `tooltip` 的需求（PR实现），可以在对文本处理后通过 `callback` 的方式进行设置，如：

```html
<div v-clamp="{
  content: detail.title,
  row: 1,
  cb: () => {}
}">
  <span>span1</span>
  <span :class="targetClass"></span>
</div>
```


## 最后

对于后续有更多需求场景，还需对该指令进一步迭代。
