---
title: this.$toast() 了解一下？
date: 2019-05-13 17:55:45
tags:
  - element-ui
  - toast
  - message
  - 组件
  - 源码
---

## 前言

在平时的开发过程中，我们总是先写好一个组件，然后在需要的页面中用 `import` 引入即可，但如果是下面这种类型的组件呢 👇
![](https://user-gold-cdn.xitu.io/2019/4/16/16a26c57558c8746?w=550&h=148&f=png&s=14688)
上面这种类型的浮层提示有一个很大的特点，就是使用频率特别高，几乎每个页面都会用到它，于是乎我们就要在每个页面中去引入该组件，并且在每个页面都得通过一个变量来控制它的显隐，这显然不是我们想要的 🙅。。。那我们想要的是什么样呢 🤔？用过一些 UI 框架的同学们应该知道有这样一种用法：

```js
this.$toast({
  duration: 3000,
  content: '这是一条消息提示'
});
```

没错，就是这么简单的一句话就万事大吉了（就是用 js 调用组件而已啦 🧐）。那这种效果究竟是怎么实现的呢？今天就让我们来（手把手 🤝 ）一探究竟吧！

<!-- more -->

## 前置知识

不知道小伙伴们有没有用过 `Vue.extend()` 这个东东，反正我是很少碰过，印象不深，所以这里我们先来短暂了解一下 `Vue.extend()` 主要是用来干嘛的。先来个官方说明（不多的，坚持下）：
![](https://user-gold-cdn.xitu.io/2019/4/1/169d96848be8777b?w=1306&h=1478&f=png&s=210393)
没怎么看懂？😴 没关系，不重要，你只要记住（加少许理解）以下用法即可：

```js
// 导入以往的普通组件
import Main from './main.vue';
// 用 Vue.extend 创建组件的模板（构造函数）
let mainConstructor = Vue.extend(Main);
// 实例化组件
let instance = new mainConstructor();
// 挂载到相应的元素上
instance.$mount('#app');
```

不知道你看懂没有，上面的 `Vue.extend(Main)` 就是一个基于 main.vue 的组件模板（构造函数），`instance` 是实例化的组件，`$mount()` 是手动挂载的意思。其中 `Vue.extend()` 和 `$mount()` 就是我们通过 js 调用、渲染并挂载组件的精髓所在，相当于早前的 `createElement` 和 `appendChild`，有异曲同工之效。这个点需要我们好好熟悉一下，所以你可以先停下来屡屡思路 🤔。
补充一下 🤐：`$mount()` 里面如果没有参数，说明组件只是渲染了但还没有挂载到页面上，如果有正确的（元素）参数则直接挂载到元素下面。

## 写一个 toast 组件

js 调用归调用，最原始的组件还是要有的，只是我们不通过 `import` 来引入到页面中而已。ok，我们就以最开始的那个 toast 图片来简单写一下这个 vue 组件（message 和 alert 也是一样的）。这里就直接上代码啦，毕竟它的结构简单到爆了，也不是本章节的重点：

```html
<!-- main.vue -->
<template>
  <div class="toast">
    <p>服务器错误，请稍后重试</p>
  </div>
</template>
<script>
  export default {
    name: 'Toast',
    mounted() {
      setTimeout(() => {
        // 3s 后通过父级移除子元素的方式来移除该组件实例和 DOM 节点
        this.$destroy(true);
        this.$el.parentNode.removeChild(this.$el);
      }, 3000);
    }
  };
</script>
<style lang="scss" scoped>
  .toast {
    display: flex;
    align-items: center;
    justify-content: center;
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    color: #fff;
    z-index: 9999;
    background: transparent;
    > p {
      padding: 12px 22px;
      font-size: 18px;
      border-radius: 4px;
      background: rgba(17, 17, 17, 0.7);
    }
  }
</style>
```

上面的内容想必大家应该都能看懂，所以这里就直接讲下面的重点了。

## 写一个 main.js

我们在 main.vue 的同级目录下新建一个 main.js 文件。我们先瞟一眼文件内容（也不多，已经是个最简版了）👇：

```js
// main.js
import Vue from 'vue'; // 引入 Vue 是因为要用到 Vue.extend() 这个方法
import Main from './main.vue'; // 引入刚才的 toast 组件

let ToastConstructor = Vue.extend(Main); // 这个在前面的前置知识内容里面有讲到
let instance;

const Toast = function() {
  instance = new ToastConstructor().$mount(); // 渲染组件
  document.body.appendChild(instance.$el); // 挂载到 body 下
};

export default Toast;
```

上面的代码暴露了一个 Toast 函数。为什么要暴露一个函数呢？原因很简单：你想想，我们最终是不是要根据 `this.$toast()` 来调用一个组件，说白了，通过 js 调用，本质就是调用一个 函数。也就是说 `this.$toast()` 就是执行了上面代码中导出的 `export default Toast`，也就是执行了 Toast 函数（`const Toast = function() {}`），所以当我们调用 `this.$toast()` 的时候其实就是执行了 `Toast()` 函数。而 `Toast()` 函数只做了一件事情：就是通过手动挂载的方式把组件挂载到 body 下面。
补充一下 🤐：一般来说我们常见的是 `$mount("#app")`，也就是把组件挂载到 `#app` 下面，`<router-view />` 也包含在 `#app` 中，但是我们这种 toast 提示是放在 `body` 下面的，也就是说它不受 `#app` 和 `<router-view />` 的管控，所以当我们切换页面（路由）的时候，这个 toast 组件是不会跟着立马消失的，这点要注意哦 😯。
这里顺便给个组件的目录结构，如下图所示：
![](https://user-gold-cdn.xitu.io/2019/4/17/16a2b5f128272155?w=418&h=578&f=png&s=47773)

## 开始调用

调用方式很简单，首先我们在入口文件 main.js（和上面不是同一个 😢） 里加上两行代码，这样我们就能在需要的地方直接用 js 调用它了，如下图所示：
![](https://user-gold-cdn.xitu.io/2019/4/17/16a294dc48c76354?w=1600&h=850&f=png&s=203640)
然后在页面中测试一下，就像下面这样子：
![](https://user-gold-cdn.xitu.io/2019/4/17/16a294f05ab491bf?w=1662&h=788&f=png&s=185432)
运行一下代码：
![](https://user-gold-cdn.xitu.io/2019/4/17/16a2956bb9437ca5?w=420&h=364&f=gif&s=3994002)
嗯，挺好，小有成就的 feel 👏👏👏。

## 支持可传参数

别急，我们好像还漏了点什么 🤔。。。对了，现在还不支持传参呢，直接调用 `this.$toast()` 就只能显示————服务器错误，请稍后重试（这下全都是后端的锅了 😊）。但我们可是个有追求的前端，不能局限于此，所以现在让我们来尝试增加下两个可配置参数，这里拿 `duration` 和 `content` 举个栗子 🌰。
首先我们要修改 main.vue 组件里面的内容（其实没啥大变化），就像下面这样：

```html
<!-- main.vue 可配置版 -->
<template>
  <div class="toast">
    <p>{{ content }}</p>
  </div>
</template>

<script>
  // 主要就改了 data
  export default {
    name: 'Toast',
    data() {
      return {
        content: '',
        duration: 3000
      };
    },
    mounted() {
      setTimeout(() => {
        this.$destroy(true);
        this.$el.parentNode.removeChild(this.$el);
      }, this.duration);
    }
  };
</script>
```

上面的代码应该算是浅显易懂了，接下来我们看下 main.js 里面改了啥：

```js
// main.js 可配置版
import Vue from 'vue';
import Main from './main.vue';

let ToastConstructor = Vue.extend(Main);

let instance;

const Toast = function(options = {}) {
  // 就改了这里，加了个 options 参数
  instance = new ToastConstructor({
    data: options // 这里的 data 会传到 main.vue 组件中的 data 中，当然也可以写在 props 里
  });
  document.body.appendChild(instance.$mount().$el);
};

export default Toast;
```

其实 main.js 也没多大变化，就是在函数里面加了个参数。要注意的是 `new ToastConstructor({ data: options })` 中的 data 就是 main.vue 组件中的 data，不是随随便便取的字段名，传入的 `options` 会和组件中的 data 合并（Vue 的功劳）。
em。。。是的，就这么简单，现在让我们继续来调用一下它：

```
<script>
export default {
  methods: {
    showToast() {
      this.$toast({
        content: "哈哈哈哈，消失的贼快",
        duration: 500
      });
    }
  }
};
</script>
```

运行一下就可以看到：
![](https://user-gold-cdn.xitu.io/2019/4/17/16a2b5b9dcd14634?w=407&h=198&f=gif&s=1100433)
当然，这还没完，我们继续添加个小功能点 🙄。。。

## 支持 this.\$toast.error()

这里我们打算支持 `this.$toast.error()` 和 `this.$toast.success()` 这两种方式，所以我们第一步还是要先去修改一下 main.vue 文件的内容（主要就是根据 `type` 值来修改组件的样式），就像下面这样：

```html
<!--main.vue-->
<template>
  <div class="toast" :class="type ? `toast--${type}` : ''">
    <p>{{ content }}</p>
  </div>
</template>
<script>
  export default {
    ...
    data() {
      return {
        type: "",
        content: "",
        duration: 3000
      };
    },
    ...
  };
</script>
<style lang="scss" scoped>
  .toast {
    ... &--error p {
      background: rgba(255, 0, 0, 0.5);
    }
    &--success p {
      background: rgba(0, 255, 0, 0.5);
    }
  }
</style>
```

其次，`this.$toast.error()` 其实就等价于 `Toast.error()`，所以我们现在的目的就是要给 Toast 函数扩充方法，也比较简单，就先看代码再解释吧：

```js
// main.js
const Toast = function(options = {}) {
 ...
};
// 以下就是在 Toast 函数中拓展 ["success", "error"] 这两个方法
["success", "error"].forEach(type => {
  Toast[type] = options => {
    options.type = type;
    return Toast(options);
  };
});
export default Toast;
```

我们可以看到 `Toast.error()` 和 `Toast.success()` 最终还是调用 `Toast(options)` 这个函数，只不过在调用之前需要多做一步处理，就是将 `["success", "error"]` 作为一个 `type` 参数给合并进 `options` 里面再传递，仅此而已 😬。
那就试试效果吧：

```html
<script>
  export default {
    methods: {
      showToast() {
        this.$toast({ content: '这是正常的' });
      },
      showErrorToast() {
        this.$toast.error({ content: '竟然失败了' });
      },
      showSuccessToast() {
        this.$toast.success({ content: '居然成功了' });
      }
    }
  };
</script>
```

![](https://user-gold-cdn.xitu.io/2019/4/17/16a2baed3a3f5811?w=345&h=324&f=gif&s=75256)

## 结语

至此，一个通过 js 调用的简单 toast 组件就搞定啦，短短的几行代码还是挺考验 js 功底的 💪。当然这只是个超简易版的 demo，显然不够完善和健壮，所以我们可以在此基础上扩充一下，比如当 `duration <= 0` 的时候，我们让这个 toast 一直显示，然后扩展一个 `close` 方法来关闭等等之类的。不过还是那句老话，实践才是检验真理的唯一标准。纸上得来终觉浅，绝知此事要躬行。step by step, day day up ! 🎉 🎉 🎉
