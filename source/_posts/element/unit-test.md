---
title: 所谓单元测试
date: 2019-07-29 09:22:38
tags:
  - element-ui
  - 单元测试
  - unit-test
---

## 前言
对于大部分前端同学们来说，可能平时都没怎么接触过单元测试🙌，顶多在初始化 Vue 项目的时候看到过它问你要不要测试，或者听说过 karma、mohca 这些名词，但具体就不得而知了。其实这东西并不复杂，只是我们没去学而已，它就像 Vue 一样容易上手，多写个几天，就能够像写 Vue 一样如鱼得水了🐠。

<!-- more -->

## 单元测试的目的
所以，我们为什么需要单元测试呢🤔？
原因很简单：就是为了减少 bug、提高产品稳定性，而不是为了测试而测试。对我们开发来说，它的好处也是显而易见的：就是保证代码质量。想想我们平时代码出问题的时候，是不是常常不敢去删除原有的代码，而是像打补丁一样往上加代码，主要原因就是没有测试保障，你也不知道自己改了对不对、影响大不大💣。所以，如果有时间的话，单元测试还是可以写写的。

## 什么是单元测试
那么，什么是单元测试呢？简单来说就是对（一些不常变动的）单元进行测试，对前端来说你可以强行理解为😁：就是对一些通用函数和通用组件进行测试。再直白点说就是写一些测试代码来验证你的源代码是否符合预期，仅此而已。
正经点说，测试又可分为测试驱动开发（TDD）和行为驱动开发（BDD）两种，什么意思呢🤔？
- TDD：通过测试来推动整个开发的进行（就是测着测着出了 bug 就返回修改代码，注重测试结果）
- BDD：通过行为来推动整个开发的进行（就是按着按着出了 bug 就返回修改代码，注重测试逻辑）

其实这些概念并不重要，我们只要了解就行，毕竟这些概念也是近几年才出现，只是个称呼。
那既然单元测试是个不错的东西，为什么大部分人都不写呢😅？说到这里，不得不说下单元测试最大的一个缺点：就是在一开始需要花很多时间。但是在大部分情况下，我们不是在写需求就是在写需求的路上，没时间搞它，所以就不懂。与之矛盾的是它的优点：就是以后可以花更少的时间😯，尤其是如果你在开发新特性时，它能大大减少副作用。
ps: 单元测试的原则就是要尽量独立和单一，这样才有利于测试、维护和理解。当然即使用例全部通过了也要经过人工测试，因为我们不能保证集成在一起就不会有问题😬。

## 前置知识（关于测试工具）
这里先抛给大家一幅测试工具的关系图：
![](https://user-gold-cdn.xitu.io/2019/7/28/16c3864cc1c12176?w=1202&h=722&f=png&s=65768)
没看懂也没关系，下面会讲解一波，但你要记住 karma 包含其他三个！！！

### karma
karma 不是一个测试框架，也不是一个断言库，而是一个测试集成工具，它的主要作用就是集成其他各种测试工具（支持按需配置，你可以通过 karma 的配置文件来集成你喜欢的框架、断言库和浏览器等），然后自动打开浏览器运行你的测试脚本，测试结果通常会显示在命令行中。此外它还可以监听测试文件的变化，然后自执行。
- 总结：你可以粗浅的认为 karma 就是用来打开浏览器的。

### mocha
mocha 是一个很常用的测试框架（类似的有 jasmine 和 jest 等），它既可以在 Node 中运行，也可以在浏览器中运行。它的主要作用是提供一些方便的语法来编写测试用例，以及对用例进行分组等。一个测试脚本可以由多个 descibe 组成，每个 describe 又可以由多个 it 组成。descibe 主要就是用来分组，it 就是具体的测试用例代码。这里简要看下它的语法，如下：
```js
describe('分组一', () => {
    it('测试用例描述一', () => {})
    it('测试用例描述二', () => {})
})
describe('分组二', () => {
    it('测试用例描述一', () => {})
    it('测试用例描述二', () => {})
})
```
这个就是固定写法，记住就行，没有什么为什么👀。
- 总结：你可以粗浅的认为 mocha 就是用来编写测试用例的。

### chai
因为 mocha 本身是不带断言的，所以需要和断言库结合使用。这里我们选择 chai 这个断言库。它有三种不同风格的写法，但意思是一样的，就像下面这样：
![](https://user-gold-cdn.xitu.io/2019/7/14/16bef71a3030c08d?w=2032&h=550&f=png&s=140659)
这里我们采用的是中间 expect 的写法，因为它比较符合自然语言（什么是自然语言？就是读起来比较顺）。然后举些例子🌰：
```js
expect(1 + 1).to.be.equal(2); // 我期待 1 + 1 等于 2
expect('hello').to.be.a('string'); // 我期待 'hello' 是个字符串
expect('').to.be.empty; // 我期待 '' 是个空值
expect({ a: 1 }).to.have.property('a'); // 我期待 { a: 1 } 有一个属性 a
```
要注意的是 chai 断言库中，to be been is has have 等这些词是没有意义的，只是为了读起来比较顺而已，事实上读起来也确实顺，如果你懂点基础英语的话。
- 总结：chai 是一个语义化的断言库

### sinon
sinon 是一个测试辅助工具，它的本质工作是测试替身，也就是用来替换测试中的部分代码，使测试代码变得简洁。比如我们要测一个函数是否被调用过，就可以借助 `sinon.fake()` 来实现，这是一个特殊的函数，现在不懂没关系，用的时候你就知道了。
- 总结：sinon 是一个测试辅助工具

以上就是单元测试所需用到的大部分工具知识，如果大家想要加深了解的话，可以自行百度。

## 开始实践
虽然花了这么大篇幅扯了这么久🌚，但上面的背景知识对我们的理解是很有帮助的。不过，好记性不如写代码，下面就让我们赶紧撸起来吧💪。

### 初始化项目
先用 vue-cli 快速生成一个最简版的 Vue 项目，这里我们选择 default。
![](https://user-gold-cdn.xitu.io/2019/7/28/16c3686e0a6a7ff2?w=582&h=132&f=png&s=18202)

### 安装各种依赖
要安装的依赖有点多，我就不详细说每个东西是干嘛的了，装就是了。
```js
yarn add karma karma-chai karma-chai-spies karma-chrome-launcher karma-mocha karma-sinon-chai mocha chai sinon sinon-chai karma-webpack vue-loader -D
```

### 新建 karma.conf.js 配置文件
执行 `./node_modules/karma/bin/karma init` 命令，一路回车，就会在根目录生成一个 karma.conf.js 配置文件。
然后对这个文件做点修改，代码如下：
```js
const VueLoaderPlugin = require('vue-loader/lib/plugin')
module.exports = function(config) {
    config.set({
        frameworks: ['mocha', 'sinon-chai', 'chai'], // 这是配置依赖包，karma 会自动引入这些包，后续我们就不需要 import 了
        files: [
            'test/**/*.test.js', // 这是要执行的测试代码
        ],
        preprocessors: { // 这是在测试之前要先用 webpack 处理一下
            "src/**/*.*": ["webpack"],
            "test/**/*.test.js": ["webpack"]
        },
        webpack: {
            mode: 'development',
            module: {
                rules: [{
                    test: /\.js$/,
                    exclude: /(node_modules)/,
                    use: [{ loader: 'babel-loader'}]
                },
                {
                    test: /\.vue$/,
                    loader: 'vue-loader'
                }]
            },
            plugins: [
                new VueLoaderPlugin()
            ]
        }
    })
})
```
顺便在根目录下新建一个空的 test 目录。
再顺便在 `package.json` 里面加上一个脚本命令 `"test": "karma start --single-run"`。
最后的目录结构大致如下：
![](https://user-gold-cdn.xitu.io/2019/7/28/16c38a918d4410af?w=416&h=654&f=png&s=60130)

## 对函数进行测试
ok，接下来让我们热个身，写个函数的测试用例。

### 写一个简单的函数
在 src 目录下新建一个 utils.js 文件，其内容如下：
```js
// utils.js
function add(a, b) {
    return a + b
}
function multiply(a, b) {
    return a * b
}
export {
    add,
    multiply
}
```

### 编写函数的测试用例
一般来说测试文件名和源码文件名是一致的，所以我们在 test 目录下新建一个 utils.test.js 文件。
```
import { add, multiply } from '../src/utils'

describe('工具函数测试', function() {
    it('求和函数测试', function() {
        let res = add(1, 1)
        expect(res).to.be.equal(2)
    })
    it('乘法函数测试', function() {
        let res = multiply(1, 1)
        expect(res).to.be.equal(1)
    })
})
```
嗯，就这样，函数用例就编写完了，当然你也可以写的再复杂点。

### 运行函数的测试用例
我们直接运行 `yarn test` 就能够看到如下结果：
![](https://user-gold-cdn.xitu.io/2019/7/28/16c373cbf0635c14?w=1796&h=450&f=png&s=168382)
可以看到我们的两个用例都通过了，也许你会问道我怎么知道它有没有运行呢，很简单，你可以把 equal 里面的值故意改成错的运行一下，形如这样：`expect(res).to.be.equal(100)`，你将会得到如下结果：
![](https://user-gold-cdn.xitu.io/2019/7/28/16c37417a46c38dd?w=2212&h=664&f=png&s=298355)
可以看到它会给你明显的错误提示。当然你也可以在 expect 后面打个 log 证明它执行了，以上就是函数的测试方法，是不是 easy 啊✌。
ps: 我们执行 `yarn test` 就是执行 `karma start --single-run`，karma 会根据 karma.conf.js 的配置内容来执行 test 目录下的代码，并自动打开浏览器测试，结束后又自动关闭浏览器（--single-run 的作用），如果有报错就会打印在控制台中。

## 对组件进行测试
接下来我们来看看 vue 组件是怎么测试的吧。首先，当然需要一个组件啦。

### 写一个简单的组件
在 src 下面新建一个简单的 demo.vue 组件，就像下面这样：
```html
<!-- demo.vue -->
<template>
    <div class="demo" :class="isError ? 'demo--error' : ''" @click="$emit('click')">
        <span class="text" :style="`opacity: ${opacity}`" :data-msg="msg">哈哈哈</span>
        <slot></slot>
    </div>
</template>
<script>
export default {
    name: 'Demo',
    props: {
        msg: {
            type: String,
            default: ''
        },
        isError: {
            type: Boolean,
            default: false
        },
        opacity: {
            type: [String, Number],
            default: 1
        }
    }
}
</script>
```

### 编写组件的基础测试用例
在 test 目录下新建 demo.test.js 文件，内容如下：
```js
import Vue from 'vue/dist/vue.common.js'
import Demo from '../src/demo.vue'

Vue.config.productionTip = false
Vue.config.devtools = false

describe('Demo 组件测试', () => {
    it('存在', () => { // 首先得确保有 demo 这个东西
        expect(Demo).to.exist // 不是 undefined、null、0、''等 fasly 值就是 exist
    })
    describe('Demo 组件的基础功能测试', () => {
        it('.text 的文本内容测试', () => {
            const Constructor = Vue.extend(Demo)
            const vm = new Constructor().$mount() // 实例化组件
            console.log(vm.$el)
            expect(vm.$el.querySelector('.text').textContent).to.equal('哈哈哈') // 我期待 .text 元素的文本内容为 '哈哈哈'
        })
    })
})
```
代码应该还算通俗易懂，其实测试用例的思路大体是一致的，主要核心思想就是：先实例化组件，然后用选择相应元素的一些可参照的东西进行断言，看看是否和预期相匹配。
ok，让我们运行 `yarn test` 看下效果：
![](https://user-gold-cdn.xitu.io/2019/7/28/16c37dfd3bde0e6c?w=2534&h=384&f=png&s=172315)
显然这个用例也是 ok 的。那如何知道错了呢，同之前的函数一样，也故意把 `equal('哈哈哈')` 改错就行，之后就不再赘述了，就像下面这样：
![](https://user-gold-cdn.xitu.io/2019/7/28/16c379546fedb828?w=2940&h=256&f=png&s=213433)

### 编写组件的 props 测试用例
我们直接上代码，大家应该都能读懂，写法是一样样的😎：
```js
// ...
describe('Demo 组件测试', () => {
    describe('Demo 组件的基础功能测试', () => {})
    describe('Demo 组件的 props 测试', () => {
        it('.text 的属性值为呵呵呵', () => { // 测试标签属性
            const Constructor = Vue.extend(Demo)
            const vm = new Constructor({
                propsData: { // 这是传参的固定写法，不必纠结
                    msg: '呵呵呵'
                }
            }).$mount()
            expect(vm.$el.querySelector('.text').getAttribute('data-msg')).to.equal('呵呵呵') // 我期待 .text 元素的 data-msg 属性值为 '呵呵呵'
        })

        it('.demo 是否有 demo--error 的样式名', () => { // 测试样式名
            const Constructor = Vue.extend(Demo)
            const vm = new Constructor({
                propsData: {
                    isError: true
                }
            }).$mount()
            expect(vm.$el.classList.contains('demo--error')).to.equal(true) // 我期待 vm.$el 的样式列表包含 demo--error 样式名
        })

        it('.text 的 opacity 样式', () => { // 测试 css 样式（放到页面中才会有样式）
            const div = document.createElement('div')
            document.body.appendChild(div)
            const Constructor = Vue.extend(Demo)
            const vm = new Constructor({
                propsData: {
                    opacity: 0.5
                }
            }).$mount(div)
            const ele = vm.$el.querySelector('.text')
            expect(getComputedStyle(ele).opacity).to.equal('0.5') // 我期待 .text 元素的 css 样式 opacity 值为 '0.5'，注意这里是字符串，css 的属性值都是字符串
        })
    })
})
```

### 编写组件的 slot 测试用例
这里也直接上代码，要注意的是 slot 和上面实例化组件的方法有点不太一样：
```js
// ...
describe('Demo 组件测试', () => {
    describe('Demo 组件的基础功能测试', () => {})
    describe('Demo 组件的 props 测试', () => {})
    describe('Demo 组件的 slot 测试', () => {
        it('slot 测试', (done) => { // 异步函数需要加 done 参数说明一下，也是固定写法
            Vue.component('xr-demo', Demo)
            let div = document.createElement('div')
            document.body.appendChild(div)
            // 这边我们的写法和上面的不太一样，不是通过 new 来实例化，而是直接写 html
            div.innerHTML = `
                <xr-demo>
                    <p id="xr"></p>
                </xr-demo>
            `
            const vm = new Vue({
                el: div
            })
            setTimeout(() => { // 这是个异步的过程，一般用 $nextTick 和 setTimeout 处理
                let p = vm.$el.querySelector('#xr')
                expect(p).to.exist // 我们期待在组件中能找到 id 为 xr 的元素
                done() // 异步函数后面需要调用一下 done()，也是固定写法
            })
        })
    })
})
```

### 编写组件的 event 测试用例
这里以 click 事件为例子🌰，那么如何测试点击事件呢？我们知道点击事件无非就是要执行一个函数，只要函数被调用了就说明点击事件发生了，那么怎么证明一个函数被执行了呢🤔？？？？嗯，是个大问题，所以，我们需要用前面说过的 `sinon.fake()` 来打辅助，具体怎么写，还是直接上代码：
```js
// ...
describe('Demo 组件测试', () => {
    describe('Demo 组件的基础功能测试', () => {})
    describe('Demo 组件的 props 测试', () => {})
    describe('Demo 组件的 slot 测试', () => {})
    describe('Demo 组件的 event 测试', () => {
        it('Demo 上的 click 事件', () => {
            const Constructor = Vue.extend(Demo)
            const vm = new Constructor().$mount()
            const callback = sinon.fake(); // 这是 sinon 的特有函数
            vm.$on('click', callback) // 添加事件监听
            vm.$el.click() // 点击组件，会触发上面👆那行的监听，从而触发 callback
            expect(callback).to.have.been.called // 我们期待 callback 被调用过
        })
    })
})
```
这个东西也是固定的套路，多写就会了，就像 Vue 一样。

## 一些问题

### 有点重复
假如你写了一遍上面的那些测试用例，你会发现代码好像有点重复，有点重复就说明我们可以优化它，于是就要说到 mocha 的几个钩子函数（这里只大概描述一下）：
```js
describe('hooks', function() {
  before(function() {
    // runs before all tests in this block
  });
  after(function() {
    // runs after all tests in this block
  });
  beforeEach(function() {
    // runs before each test in this block
  });
  afterEach(function() {
    // runs after each test in this block
  });
  // test cases
  it('case one', () => {})
  it('case two', () => {})
});
```
也就是说我们在执行 it 之前会先调用 beforeEach 这个钩子，执行 it 之后调用 afterEach 这个钩子。这样一来我们就可以把实例化组件的代码抽离出来写在 beforeEach 里面。

### 没有及时销毁
另外，你可能还注意到，我们的实例没有及时销毁，所以我们也可以在 afterEach 这个钩子里面做相应的处理，就像下面这样：
```js
afterEach(function() {
    // 移除元素并释放内存
    vm.$el.remove()
    vm.$destroy()
});
```

### 每次修改都要手动执行脚本
我们可不可以保存的时候就自动执行 `yarn test` 呢。嗯，是可以的，小小修改一下最初的脚本命令就行，就像这样：`"test": "karma start"`，这下我们保存的时候它就会自动测试一遍了。

## 别人家的单元测试
👌，接下来就是见证奇迹的时刻。现在让我们打开 Element 的源码来看看别人的单元测试是怎么写的（瞟一眼就行）：
![](https://user-gold-cdn.xitu.io/2019/7/28/16c38d2a2419f4fb?w=1840&h=1212&f=png&s=329352)
![](https://user-gold-cdn.xitu.io/2019/7/28/16c38d4f372f492a?w=1398&h=1414&f=png&s=240523)
有没有发现，你突然一下变的牛逼了，以前看不懂的东西，现在刚接触就看懂了。是的，单元测试不过如此，你要是再多写几天就可以信手拈来了（其实还是有坑的😂，但是多写就好了）。另外其实 Vue 官方已经提供了整套的测试流程，我们只需要直接编写脚本就行，不用太多准备。

## 结语
所以，怎么应用到实际工作中呢，我想大部分公司的后台管理系统应该是一个施展才华的好地方。至于覆盖率，多写多覆盖罗，对于大部分前端同学来说不必太较真，毕竟我们是要写需求的啊。最后，其实很多东西都不难，只是你没碰过所以总觉得遥不可及。常言道会者不难，难者不会，说的就是这个道理。（大赞无疆👍👍👍。。。）

ps: 如有需要上述代码的请点击这里： [github 传送门](https://github.com/lgq627628/unit-test)
