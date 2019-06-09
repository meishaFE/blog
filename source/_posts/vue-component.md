---
title: 一个弹窗内容引发的"血案"(上)
date: 2019-06-10
tags:
- Vue
- JavaScript
- Component
---

炎炎夏日、蝉鸣蛙叫、一切都变得"躁动"起来了。作为一名优秀的"代码搬运工"在这燥热的日子里也慢慢开始"钻研"一些比较无奈的"牛角尖"。今天就给大家带来前段时间我所"钻"过的"牛角尖儿"。倘若文章内容引起您的重度不适，还请海涵。

<!-- more -->

# 一、背景

实际业务需求如下图:
![需求图](https://cdn.meishakeji.com/msedu/course/cover/2019-06/b84abc7a8636ce00.png)

一个简简单单的弹窗提示操作为何成为我这次文章的重点呢？用过`Element UI`的同学肯定知道有两种组件都是用来做弹窗提示的。没错，就是`Message Box`和`Dialog`。事情到这里貌似已经没有了后续，但是这不符合我"卖关子"的风格。下面请听我慢慢道来。

# 二、寻求真理之路

首先明确一点:这次弹窗里面要用到弹出框`Popover`，也就是当鼠标悬浮至"什么是绑定"这句文字上时要弹出一张示意图。那么重点来了，我可以很明确的告诉你官方`Message Box`组件的例子中全是纯文字展示类型，没有像这样的还有交互的示例。那怎么办？用`Dialog`呗。确实可以通过`Dialog`的自定义插槽去实现。但是"浮躁"的我是绝对不会因为一个展示类弹窗效果而去引入一个`Dialog`组件，然后多写几个函数来实现我想要的效果。我就是喜欢`Message Box`这种简单粗暴的链式处理方式。

```javascript
// 官方示例
this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
  confirmButtonText: '确定',
  cancelButtonText: '取消',
  type: 'warning'
}).then(() => {
  this.$message({
    type: 'success',
    message: '删除成功!'
  });
}).catch(() => {
  this.$message({
    type: 'info',
    message: '已取消删除'
  });
});
```

于是我又仔细阅读了官方所展示出来的例子，发现它可以自定义内容。`message`属性值类型可以接受`String/VNode`。代码如下:

```javascript
const h = this.$createElement;
this.$msgbox({
  title: '消息',
  // 重点++++++++++++++++++++
  message: h('p', null, [
    h('span', null, '内容可以是 '),
    h('i', { style: 'color: teal' }, 'VNode')
  ]),
  // 重点++++++++++++++++++++
  showCancelButton: true,
  confirmButtonText: '确定',
  cancelButtonText: '取消',
}).then(action => {
  this.$message({
    type: 'info',
    message: 'action: ' + action
  });
});
```

咋一看，诶，有救了。仔细一看，完了。贴上我的业务代码，你们自行体会一下。

```javascript
let htmlStr = `<div><div style="color: #606266; font-size: 14px; margin-bottom: 5px;">
<div>家长将收到缴费通知，确认发送吗？</div>
<div style="font-size: 12px; color: #909399;">仅关注XX公众号且绑定了账号的家长会收到通知</div>
</div>
<el-popover
placement="right-start"
width="245"
trigger="hover"
>
<div style="font-size: 12px; color: #909399;">
  <div>绑定指的是家长点击梅沙H5相关链接后，输入了手机号码及验证码完成绑定，如下图所示：</div>
  <img style="width: 225px;" src="${bindPic}" alt="绑定示意图"/>
</div>
<span style="font-size: 12px; color: #909399;" slot="reference">
  <i class="iconfont icon-ic_help" style="font-size: 14px;"></i>
  <span>什么是绑定</span>
</span>
</el-popover></div>`
```

。。。这。。。
都9102年了，我还得用最原始的`DOM节点对象树`(这个名字瞎取得，别介意哈)写法去实现，不，我选择放弃。接着我又找寻新的解决方案——`$createElement`这个方法的其他用法。因为这个方法返回的是一个`VNode`正是我所需要的，只不过目前这种书写方式我不接受。于是，我在`Vue`官方文档找到了这个方法的介绍，惊人的发现它可接受的参数如下:

```javascript
// {String | Object | Function}
// 一个 HTML 标签名、组件选项对象，或者
// resolve 了上述任何一种的一个 async 函数。必填项。
```

组件选项对象？这是啥？起初我也很懵逼，我以为是一个`Vue`的实例，但是当我传入一个实例之后显然是不成功的。瞬间我寻求真理的步伐又被迫停下。出于无奈，先把这个需求撂下了，开始开发别的需求功能，但当我新建一个组件时，这个`.vue`文件的模样让我"茅厕顿开"。给你们感受下:

```javascript
<template>
  <section>
  </section>
</template>

<script>
export default {
  name: '',
  data () {
    return {
    };
  },
  components: {
  },
  created () {
  },
  mounted () {
  },
  methods: {
  }
};
</script>

<style lang="scss" scoped>
</style>
```

单个组件文件所`export`出去的就是一个对象，那么是不是就是这个玩意儿呢？实践出真知，我立马将我上面的业务代码封装成一个组件然后`import`进来，最后一试，居然成了。代码如下:

```javascript
import myComponent from 'xxx';

export default {
  // ....
  methods: {
    renderFunc () {
      const h = this.$createElement;
      this.$msgbox({
        title: '消息',
        // 重点++++++++++++++++++++
        message: h(myComponent),
        // 重点++++++++++++++++++++
        showCancelButton: true,
        confirmButtonText: '确定',
        cancelButtonText: '取消',
      }).then(action => {
        this.$message({
          type: 'info',
          message: 'action: ' + action
        });
      });
    }
  }
}
```

后面继续深挖了一下(真的就一下)，才知道原来所谓的组件选项对象就是组件的配置对象也就是我们在单个组件文件(`.vue`)所暴露出来的包含`name、data、methods、created`等等配置组件属性的对象；也就是我们实例化一个组件——`new Vue(options)`中的这个`options`；也就是我们一个组件实例`vm`下挂载的`$options`属性。所以上面我往`$createElement`中传入一个实例是本末倒置了！

# 三、后续

到这里似乎我的真理之路走到了尽头，但是如果这个没有任何复用价值的内容，我也要单独用一个`.vue`文件去实现很显然是非常愚蠢的做法！所以我是坚决不会这么做的。所以，我又开始了我的"征程"。由于时间和文章篇幅的关系，我将推出"血案"的第二卷——讲叙我认为的"终极方案"。

***注:作者才疏学浅，如有不当的地方欢迎指出！***