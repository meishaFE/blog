---
title: 多行文本省略实现攻略
date: 2019-12-16
tags:
- ellipsis
---

转眼一年又过去了，似乎今年的前端圈“格外的安静”，没了以往的“惊天动地”、“改天换地”、“翻天覆地”。给作者的感觉就像是前几年突然燥起来的民谣一样慢慢地“沉”了下去，所以作者曾一度认为“切图仔”是一群“高逼格，低收入”的“文艺民谣狗”。不过也许我们可能都在“酝酿”着“大事情”。既然没了那么大的技术写，那么今天就跟随作者在***南京李先生***的背景音乐下谈谈多行文本省略的小技巧。

<!-- more --->

## 一、单行文本

俗话说的好“欺软怕硬”，“柿子专找软的捏”，我们也先从最简单的单行文本省略说起。首先要实现单行文本省略，不管是前端老司机还是前端小白都能默写出如下的代码：

```css
.ellipsis-text {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  width: 200px;
}
```

这种简单粗暴的业界默认“规范”在这就不多赘述。我们直接看升级版——不仅要省略掉超出的内容，还要实现在用户鼠标移至这段内容时显示出完整的文本内容。

如何实现呢？基础功扎实的同学可能立马就想到了`HTML`标签似乎有个`title`的属性可以实现这种效果，如下：

```html
// HTML
<div class="ellipsis-text" title="世上纵有千万曲，歌坛再无李先生——“缅怀”南京李先生">世上纵有千万曲，歌坛再无李先生——“缅怀”南京李先生</div>
```

效果图：![效果图1](https://cdn.meishakeji.com/msedu/course/cover/2019-12/5a82caeb06531d14.jpeg)

这种的话首先用户需要停留个几秒钟才能显示出来，还不能复制内容，而且样式也无法很灵活地弄成我们想要的。所以这种方式‘out’。

```javascript
// HTML
<el-tooltip
  :disabled="!isShowToolTips"
  content="世上纵有千万曲，歌坛再无李先生——“缅怀”南京李先生"
  placement="top-start"
  effect="light"
>
  <div
    class="ellipsis-text"
    @mouseenter.self="handleMouseEnter"
  >世上纵有千万曲，歌坛再无李先生——“缅怀”南京李先生</div>
</el-tooltip>

// JS
export default {
  name: 'DemoCom',
  data () {
    return {
      isShowToolTips: false
    }
  },
  methods: {
    handleMouseEnter (e) {
      let dom = e.target;
      let targetWidth = dom.clientWidth;
      let fontSize = +getComputedStyle(dom, null).fontSize.replace('px', '');
      this.isShowToolTips = this.checkTextWidth(targetWidth, '世上纵有千万曲，歌坛再无李先生——“缅怀”南京李先生', fontSize);
    },
    // 满足超过一行显示省略号，并展示tooltips
    checkTextWidth (targetWidth, textContent, fontSize) {
      let dom = document.createElement('span');
      dom.setAttribute('style', `display: inline-block; white-space: nowrap; font-size: ${fontSize}px`);
      dom.innerHTML = textContent;
      document.body.appendChild(dom);
      let realWidth = dom.getBoundingClientRect().width;
      document.body.removeChild(dom);
      return realWidth > targetWidth;
    };
  }
}
```

效果图：![效果图2](https://cdn.meishakeji.com/msedu/course/cover/2019-12/9f9a0a304595ae9e.png)

这里我是用了`Element UI`的`el-tooltip`实现的提示，同学们可根据各自的实际技术栈选用不同的组件库。关于实现思路，请仔细看代码，**你品，你细品**。

## 二、多行文本

好，讲完了简单的，来点复杂的。同样如果是基础功扎实的且涉及过移动端项目的同学应该就知道在微信的公众号H5中用几行`css`样式代码就可以实现，如下：

```css
.doubleline-ellipsis {
  overflow : hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  word-break: break-all;
  -webkit-line-clamp: 2; // 2 => 对应的行数
  /* autoprefixer: off */
  -webkit-box-orient: vertical; // 解决这个属性webpack无法识别，移除的问题
  /* autoprefixer: on */
}
```

如果不做其他花里胡哨的交互这几行代码就足以满足极大部分H5页面了。(基本上大部分H5所运行的主流浏览器内核都是`webkit`所以你懂的)但是很明显这不符合作者“爱搞事”的风格。所以会去想怎么样做到可以通过属性配置来控制显示的文本行数呢？怎么在其他类型内核的浏览器中实现同样的效果呢？如果我要加一个展开、收起按钮呢？

首先针对在别的浏览器内核实现同样的效果(单纯的超出显示省略号),网上攻略一大把，什么伪类覆盖，什么截断加`...`等等，这里就不啰嗦了，大家自力更生。

我们主要讲解如何实现如下加展开收起的效果：![效果图3](https://cdn.meishakeji.com/msedu/course/cover/2019-12/ce24fde292f668e2.png)

如果是在H5下实现，那么我们就可以利用上面说的方法偷个懒，代码如下：

```javascript
// HTML
<div
  :class="{'doubleline-ellipsis': !isExpand}"
>XXXXXXXXX</div>
<div
  v-if="isMultiple"
  @click="() => isExpand = !isExpand"
  class="expand-btn"
>{{isExpand ? '收起' : '展开'}}</div>

// JS
export default {
  name: 'Demo',
  data () {
    return {
      isMultiple: false,
      isExpand: false
    }
  },
  mounted () {
    this.computedWidth();
  },
  methods: {
    computedWidth () {
      let dom = this.$refs.courseRateItem;
      let domPadding = +getComputedStyle(dom.children[1]).paddingLeft.replace('px', '') * 2; // 内边距
      let fontSize = 14 / 37.5; // 字体大小
      if (!dom) return;
      this.isMultiple = $tool.checkTextWidth((dom.clientWidth - domPadding) * 2, this.rateMsg.content, fontSize);
    },
    checkTextWidth (targetWidth, textContent, fontSize) {
      let dom = document.createElement('span');
      dom.setAttribute('style', `display: inline-block; white-space: nowrap; font-size: ${Math.round(fontSize * 100) / 100}rem`);
      dom.innerHTML = textContent;
      document.body.appendChild(dom);
      let realWidth = dom.offsetWidth;
      let newFontSize = Math.round(getComputedStyle(dom).fontSize.replace('px', ''));
      document.body.removeChild(dom);
      return realWidth > Math.round(targetWidth) ||
      Math.round(targetWidth) - realWidth === newFontSize ||
      ((realWidth < Math.round(targetWidth)) && Math.round(targetWidth) - realWidth < newFontSize / 2);
    };
  }
}
```

简单讲几点：

- 注意字体像素单位的转换(px <=> rem)
- `realWidth > Math.round(targetWidth) ||
  Math.round(targetWidth) - realWidth === newFontSize ||
  ((realWidth < Math.round(targetWidth)) && Math.round(targetWidth) - realWidth < newFontSize / 2)`这段代码判断的意思：
  - `realWidth > Math.round(targetWidth)`实际内容宽度远大于目标宽度
  - `Math.round(targetWidth) - realWidth === newFontSize`实际内容宽度于目标宽度正好相差一个字体像素大小
  - `((realWidth < Math.round(targetWidth)) && Math.round(targetWidth) - realWidth < newFontSize / 2)`实际内容宽度比目标宽度至少小于半个字体像素大小(针对英文等其他特殊字符)

其实最后一个判断是作者被逼无奈投机取巧的方法，通过大量地“实验”总结出了这个规则，但实际对于一些极其特殊或者不规则的字符还是会出现精度误差问题。但是大部分情况还是满足的。

到这就结束了？不可能，我们继续搞事。由于作者水平有限想不出其他更好的方法，所以开始另觅出路，充分发挥程序员的“开源思想”。于是找到了鼎鼎大名的`Antd-design`来学习它的实现思想。

先来一份关键代码：

```javascript
function inRange() {
  return ellipsisContainer.offsetHeight < maxHeight;
}
function measureText (textNode, fullText, startLoc = 0, endLoc = fullText.length, lastSuccessLoc = 0) {
  const midLoc = Math.floor((startLoc + endLoc) / 2);
  const currentText = fullText.slice(0, midLoc);
  textNode.textContent = currentText;

  if (startLoc >= endLoc - 1) {
    // Loop when step is small
    for (let step = endLoc; step >= startLoc; step -= 1) {
      const currentStepText = fullText.slice(0, step);
      textNode.textContent = currentStepText;

      if (inRange()) {
        return step === fullText.length
          ? {
              finished: false,
              reactNode: fullText,
            }
          : {
              finished: true,
              reactNode: currentStepText,
            };
      }
    }
  }

  if (inRange()) {
    return measureText(textNode, fullText, midLoc, endLoc, midLoc);
  }
  return measureText(textNode, fullText, startLoc, midLoc, lastSuccessLoc);
}
```

简单说几点：

- 该UI库是从高度的方向入手来进行溢出判断的
- 采用算法使用频率最高的”二分法“来进行逐一分割字符串
- 方式基本上适用所有场景，解决了上面投机取巧方式无法在不支持`-webkit-line-clamp`属性的浏览器中实现的问题
- 不知道存不存一些临界判断误差的问题

总结下来，大厂的东西真香！

## 三、总结

好了，趁着那一句“这个世界会好吗？”的余音未散，让我们一起带着今天的内容深刻缅怀下“南京李先生”。拜谢！

*PS:个人理解能力与研究深度有限，故文章中有任何不妥，欢迎指出！*
