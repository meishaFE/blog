---
title: 复杂公式渲染初探
date: 2019-09-16 22:02:00
tags:
- js库
---

最近由于项目原因开始接触到latex格式的渲染，主要是数学、物理公式等，渲染方式是html展示，但是每个公式都重新手写html和样式肯定不太现实，于是就接触到了两个能渲染latex的库。
<!-- more -->

# latex
>LaTeX， 是一种基于TEX的排版系统，由美国电脑学家莱斯利·兰伯特在20世纪80年代初期开发，利用这种格式，即使用户没有排版和程序设计的知识也可以充分发挥由TEX所提供的强大功能，能在几天，甚至几小时内生成很多具有书籍质量的印刷品。对于生成复杂表格和数学公式，这一点表现得尤为突出。因此它非常适用于生成高印刷质量的科技和数学类文档。这个系统同样适用于生成从简单的信件到完整书籍的所有其他种类的文档。
## MathJax
1. MathJax的[官网地址](https://www.mathjax.org)，中文文档地址[点这里](https://mathjax-chinese-doc.readthedocs.io/en/latest/index.html)
### 引入MathJax
在使用MathJax之前，需要通过CDN引入, 在`<body>`标签中添加：
```js
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.0.0/MathJax.js?config=TeX-MML-AM_CHTML"></script>。
```
### 配置MathJax
将MathJax的配置封装成一个函数：
```js
let isMathjaxConfig = false; // 防止重复调用Config，造成性能损耗

const initMathjaxConfig = () => {
  if (!window.MathJax) {
    return;
  }
  window.MathJax.Hub.Config({
    showProcessingMessages: false, //关闭js加载过程信息
    messageStyle: "none", //不显示信息
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
      inlineMath: [["$", "$"], ["\\(", "\\)"]], //行内公式选择符
      displayMath: [["$$", "$$"], ["\\[", "\\]"]], //段内公式选择符
      skipTags: ["script", "noscript", "style", "textarea", "pre", "code", "a"] //避开某些标签
    },
    "HTML-CSS": {
      availableFonts: ["STIX", "TeX"], //可选字体
      showMathMenu: false //关闭右击菜单显示
    }
  });
  isMathjaxConfig = true; // 
};
```
### 使用MathJax渲染
MathJax提供了`window.MathJax.Hub.Queue`来执行渲染。在执行完文本获取操作后，进行渲染操作：
```js
if (isMathjaxConfig === false) { // 如果：没有配置MathJax
  initMathjaxConfig();
}

// 如果，不传入第三个参数，则渲染整个document
// 因为使用的Vuejs，所以指明#app，以提高速度
window.MathJax.Hub.Queue(["Typeset", MathJax.Hub, document.getElementById('app')]);
```
## katex 号称最快的数学公式渲染库
katex是可汗学院推出的一种解析latex的库，具有简单的api、无依赖、高效、快速、服务器端渲染的js解析库；
特点：
1. 简单的API：不依赖其它 javascript库；
2. 快速：KaTeX 是将其数学同步且不需要回流页；
3. 输出质量：KaTeX 的布局是基于 Donald Knuth 的 Tex ，数学排版的黄金标准；
4. 服务器的渲染：无论浏览器或环境如何，KaTeX 输出保持一致，因此可使用 Node.js 预渲染表达式，并将其作为纯 HTML 发送。
5. 该库只支持纯粹的latex格式的渲染，如果包含html实体、或者标签将渲染失败

[官网地址](https://katex.org)
### 引入katex
1. 通过cdn引入
```html
<!DOCTYPE html>
<!-- KaTeX requires the use of the HTML5 doctype. Without it, KaTeX may not render properly -->
<html>
  <head>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.11.0/dist/katex.min.css" integrity="sha384-BdGj8xC2eZkQaxoQ8nSLefg4AV4/AwB3Fj+8SUSo7pnKP6Eoy18liIKTPn9oBYNG" crossorigin="anonymous">

    <!-- The loading of KaTeX is deferred to speed up page rendering -->
    <script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.0/dist/katex.min.js" integrity="sha384-JiKN5O8x9Hhs/UE5cT5AAJqieYlOZbGT3CHws/y97o3ty4R7/O5poG9F3JoiOYw1" crossorigin="anonymous"></script>

    <!-- To automatically render math in text elements, include the auto-render extension: -->
    <script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.0/dist/contrib/auto-render.min.js" integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI" crossorigin="anonymous"
        onload="renderMathInElement(document.body);"></script>
  </head>
  ...
</html>
```
或者用npm或yarn
```
npm install katex
# or globally
npm install -g katex

yarn add katex
# or globally
yarn global add katex
```
## API
### katex.render
该方法会将公式渲染的结果返回到elemnet里
```js
// 官方实例
katex.render("c = \\pm\\sqrt{a^2 + b^2}", element, {
    throwOnError: false
});
// 如果需要转义那么可以这样
katex.render(String.raw`c = \pm\sqrt{a^2 + b^2}`, element, {
    throwOnError: false
});
请注意本地复制代码测试的时候要去掉转义字符,不然会出现错误
```
### katex.renderToString
该渲染方法会将生成的dom结构输出到html

```js
var html = katex.renderToString("c = \\pm\\sqrt{a^2 + b^2}", {
    throwOnError: false
});
```
### renderMathInElement
插件的使用，引入插件的js后，会获得一个全局的函数renderMathInElement
```js
document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
            // ...options...
        });
    });
```
该函数提供两个参数，第一个提供dom节点就是说会渲染该dom节点下的公式元素（尽可能的精确会提高性能），第二个参数是分隔符配置文件，将分隔符以内的字符串会进行katex渲染，配置如下
```js
// display设置为true会被渲染成块元素，false是行内元素
 {
  delimiters: [
    {left: "$$", right: "$$", display: false},
    {left: "\[", right: "\]", display: true},
    {left: "$",  right: "$",  display: false},
    {left: "\(", right: "\)", display: false }
  ]
}
```
这些就是katex的基本api
## 总结
不管是render、还是renderToString都只支持纯公式的渲染一旦包含标签或者其他不是公式的元素字符串那么就会渲染失败，所以要是渲染带标签的就需要引入renderMathInElement，然后在公式左右设置分隔符来明确表示出公式，要不然就会出现“乱码”，类似/dfrac{a}{b}，t_{a} neq t_{b}这种。