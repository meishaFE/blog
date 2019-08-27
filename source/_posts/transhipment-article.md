---
title: (转)前端工程师如何持续保持热情
date: 2019-08-27
tags:
- 转载、杂文
---

憋了许久，想着输出点啥东西，捣鼓半天依旧没有任何思绪，可能肚子里没墨水了吧。看来是时候疯狂学习一波了，所以今天转载一篇来自[掘金lhyt](https://juejin.im/user/5acc4ab4f265da239148703d)的文章，喝喝"鸡汤"。原文地址：[https://juejin.im/post/5d6419dee51d4561eb0b26af?utm_source=gold_browser_extension](https://juejin.im/post/5d6419dee51d4561eb0b26af?utm_source=gold_browser_extension)。

<!-- more -->

> 对于一种事情，经常重复的话，很容易就会厌烦、觉得无趣、失去了当初的热情。

* 做不完的业务需求，日复一日，就觉得工作乏味、都是体力活；
* c端做多了，就觉得业务逻辑没有挑战性，没意思，设计要求苛刻，特别烦；
* b端做多了，就觉得天天写平台，天天对着无味的数据，没机会玩一下炫酷的特效；
* 技术建设做多了，看着自己做的东西都腻了；
* 研究一些花哨的东西，又对工作内容没有什么意义；
* 想用一下最新技术，然而项目历史原因又望洋兴叹......

自然而然，就失去了当初的热情，找不到成就感，甚至还怀疑，自己是不是不适合做前端，是不是应该换一份工作，是不是要转行了？

# 避免重复用同样的方法做同样的事情

如果一直以同样的姿态做一样的事情，就很容易觉得无聊，没有成就感。所以需要提升效率做同样的事情，后面越来越快完成，每天都看见自己的进步，自然就有了热情

## 精简代码，提高代码质量

> 当你要copy的时候，就要想一下哪里可以封装、哪里要抽离逻辑、要复用哪里的代码了

### 不要让一块基本差不多的代码重复存在

大家入门的时候，可能写过这样的代码：

```javascript
<a>首页</a>
<a>关于我们</a>
<a>合作</a>
<a>加入我们</a>
```

后来发现，vue可以v-for，react可以map，原生可以循环插入fragment里面最后一次性append。

比较明显的大家可以发现，如果不太明显又能复用的我怎么发现呢？还是那句话，当你copy的时候，代码就能复用。举个antd Form常见的一个应用场景：

```javascript
<Form>
  <Item label="名称">
  {getFieldDecorator('name', {
    rules: [{
      required: true,
      message: '请输入名称',
    }],
  })(<Input />)}
  </Item>
  <Item label="描述">
  {getFieldDecorator('desc', {
    rules: [{
      required: true,
      message: '请输入描述',
    }],
  })(<Input />)}
  </Item>
  <Item label="类型">
  {getFieldDecorator('type', {
    rules: [{
      required: true,
      message: '请选择类型',
    }],
  })(<Checkbox />)}
  </Item>
</Form>
```

套路都是一样的，结构也是一样，那我们就维护一个配置对象来维护这个form：

```javascript
const items = [
  {
    label: '名称',
    key: 'name',
    decorator: {
      rules: [{
      required: true,
      message: '请输入名称',
    }],
    },
    component: <Input />
  },
  // ...
]
```

再比如一个很长的if，后台错误码在前端处理，很常见的一段代码：

```javascript
// before
  if (type === 1) {
    console.log('add')
  } else if (type === 2) {
    console.log('delete')
  } else if (type === 3) {
    console.log('edit')
  } else {
    console.log('get')
  }
  // after
  const MAP = {
    1: 'add',
    2: 'delete',
    3: 'edit',
    4: 'get',
  }
  console.log(MAP[type])
```

> 通过配置对象、循环渲染来减少重复代码

### 要有一种“懒得写代码”的心态

比如redux的action type

```javascript
const FETCH_LIST = 'FETCH_LIST'
const FETCH_LIST_SUCCESS = 'FETCH_LIST_SUCCESS'
const FETCH_LIST_FAILED = 'FETCH_LIST_FAILED'
const FETCH_USERINFO = 'FETCH_USERINFO'
const FETCH_USERINFO_SUCCESS = 'FETCH_USERINFO_SUCCESS'
const FETCH_USERINFO_ERROR = 'FETCH_USERINFO_ERROR'
```

很整齐又看起来很舒服的代码，但是它们都有共性，异步请求，请求中、请求成功、请求失败的type。每次新增，我们先来这里复制三行，再改改。既然都差不多，我们可以写个type生成器：

```javascript
function actionGenerator(k = '') {
  const key = k.toUpperCase()
  return {
    ...(k
      ? {
        [`FETCH_${key}`]: `FETCH_${key}`,
        [`FETCH_${key}_SUCCESS`]: `FETCH_${key}_SUCCESS`,
        [`FETCH_${key}_ERROR`]: `FETCH_${key}_ERROR`,
      }
      : {}),
  };
}
// 从此以后，action_type代码行数大大减少
```

再比如一个函数里面对一个对象反复赋值操作：

```javascript
// before
obj.a = 1
obj.b = 2
obj.c = 5
// after
const newVals = {
  a: 1,
  b: 2,
  c: 5
}
// 如果业务里面的obj很依赖原本引用，不能改变原对象
Object.keys(newVals).forEach(key => {
  obj[key] = newVals[key]
})
// 如果业务里面的obj不依赖原本引用，可以改变原对象
obj = { ...obj, ...newVals}
// 以后要改什么，我只要去改一行newVals就可以
```

再比如页面文案，我们可以单独拎出去到一个文件里面统一配置，以后修改很方便

```javascript
// before
<header>练习不足两年半的练习生</header>
<section>我只是一个练习生</section>
<ul>
  <li>唱</li>
  <li>跳</li>
  <li>rap</li>
</ul>
<footer>联系方式：000</footer>

// after
const CONSTANT = {
  title: '练习不足两年半的练习生',
  desc: '我只是一个练习生',
  hobbies: ['唱', '跳', 'rap'],
  tel: '000'
}
<header>{CONSTANT.title}</header>
<section>{CONSTANT.desc}</section>
<ul>
  {
    CONSTANT.hobbies.map((hobby, i) => <li key={i}>{hobby}</li>)
  }
</ul>
<footer>联系方式：{CONSTANT.tel}</footer>
```

这是一个看起来好像写了更多代码，变复杂了。一般情况下，是不需要这样的。**对于运营需求**，这种方案应付随时可以变、说改就要改的文案是轻轻松松，而且还不需要关心页面结构、不用去html里面找文案在哪里，直接写一个文件放CONSTANT这类东西的。而且这个对象还可以复用，就不会有那种“改个文案改了几十个页面”的情况出现。

还有一个场景，我们平时可能写过很多这样的代码：

```javascript
function sayHi(name, word) {
  console.log(`${name}: ${word}`)
}
const aSayHi = () => sayHi('a', 'hi')
const aSayGoodbye = () => sayHi('a', 'goodbye')
const aSayFuck = () => sayHi('a', 'fuck')
```

当然这是很简单的场景，如果sayHi函数传入的参数有很多个，而且也有很多个是重复的话，代码就存在冗余。这时候，需要用**偏函数**优化一下：

```javascript
const aSay = (name) => sayHi('a', name)
const aSayHi = () => aSay('hi')
const aSayGoodbye = () => aSay('goodbye')
const aSayFuck = () => aSay('fuck')
```

三元、短路表达式用起来，举几个例子

```javascript
// before
if (type === true) {
  value = 1
} else {
  value = 2
}
//after
value = type ? 1 : 2

// before
if (type === DEL) {
  this.delateData(id)
} else {
  this.addData(id)
}
// after
this[type === DEL ? 'delateData' : 'addData'](id)
// or
;(type === DEL ? this.delateData : this.addData)(id)

// before
if (!arr) {
  arr = []
}
arr.push(item)
// after 这个属于eslint不建议的一种
;(arr || (arr = [])).push(item)

// before
if (a) {
  return C
}
// after
return a && C
```

最后一个例子，比如一个重复的key赋值过程，可以用变量key简化

```javascript
switch(key) {
  case 'a':
  return { a: newVal }
  case 'b':
  return { b: newVal }
  case 'c':
  return { c: newVal }
}
// after
return { [key]: newVal }
```

**however, 无论当时多熟悉，代码写得多好，也必须写注释。** 核心算法、公共组件、公共函数尤其需要注释

```javascript
/**
  * 创建树状组织架构
  * @param {Object} orgs
  * @param {Array} [parent=[]]
  * @param {Boolean} check 是否需要校验
  */
```

> 小结：代码简短，没有重复，自己看了也不会腻；抽离逻辑，封装公共函数，无形提高代码质量。最后带来的效益是，加快了开发效率，也提升开发体验与成就感：“终于不是天天copy了，看着自己一手简单优雅的代码，越来越想做需求了”

## 如何让运营需求不枯燥无味

大部分公司都是以业务需求为主，除了自己家的主打产品迭代需求外，另外的通常是运营需求作为辅助。运营类需求，通常具有短时效性、频率高、有deadline的特点。

一个运营活动，可能有多种模式、多种展示布局、多种逻辑，产品运营会随机组合，根据数据来调整寻求最佳方案，这就是场景的运营打法——ab test。

有的人面对这种情况，总是会感叹：又改需求了、天天改又没什么用、怎么又改回来了、你开心就好。其实这种情况，我们只要给自己规划好未来的路，后面真的随意改，甚至基本不需要开发

我们从一个简单的用户信息页面的例子入手：

```javascript
render() {
  const { name, score } = this.state.info
  return (
    <main>
      <header>{name}</header>
      分数：<section>{score}</section>
    </main>
  )
}
```

### 增加适配层

我们尽量不要动核心公共逻辑代码，不要说"只是加个if而已"。引入新的代码，可能会引入其他bug（常见错觉之一： 我加一两行，一定不会造成bug），有一定的风险。

回到主题，如果突然要说这个活动要拉取另一次游戏的分数，直接去把请求改了，然后再把组件所有的字段改了，当然是一个方法。如果改完不久，又要说改回去，那是不是吐血了。显然我们需要一个适配层：

```javascript
function adapter(response, info) {
  return Object.keys(info).reduce((res, key) => {
    res[key] = response[info[key]]
    return res
  }, {})
}
// before
function fetchA() {
  return request('/a')
}
fetchA.then(res => {
  this.setState({
    info: { name: res.nickname, score: res.counts }
  })
})

// after 直接修改原请求函数
function fetchA() {
  return request('/a').then(res => {
    // 把适配后的结果返回，对外使用的时候不变
    return adapter(res, {
      name: 'nickname',
      score: 'counts'
    })
  })
}
```

我们把适配集中到一起了，每次要改，只要来这里改适配层、改请求接口即可。当然，后台如果全部统一的话是最好，只是有历史原因不是说改就改的。

> 拓展： 如果改来改去的接口很多呢？ 此时，我们需要维护一个配置表，保存某个请求与适配对象，而不是直接去把之前的代码改掉

```javascript
const cgiMAp = {
  '/a': {
    name: 'nickname',
    score: 'counts'
  },
  // ...
}
```

通过读取这个配置，在请求函数里面封装一个读取逻辑，即可适配所有的相关接口。后面如果换了一种数据来源渠道，那也光速解决需求

```javascript
function reqGenerator(cfg) {
  return Object.keys(cfg).reduce((res, key) => {
    res[key.slice(1)] = () => request(key).then(r => adapter(r, cfg[key]))
    return res
  }, {})
}
const reqs = reqGenerator(cgiMAp)
reqs.a()
```

### 严格遵守组件化

> 对于前面的那个用户信息页面的例子，如果用户信息页面需要填更多的内容呢，如果不想展示分数呢？

这种情况，先要和产品侧确认，哪些内容是以后必须在的，哪些是会变的。会多变的部分，我们直接使用content读进来：

```javascript
class App extends Component {
  // ...
  render() {
    const { name, score } = this.state.info
    return (
      <main>
        <header>{name}</header>
        <section>{this.props.children}</section>
        分数：<section>{score}</section>
      </main>
    )
  }
}
```

这样子，组件上层只要包住个性化组件内容就ok

```javascript
<App>
  <section>我只是一个练习生</section>
  <ul>
    <li>唱</li>
    <li>跳</li>
    <li>rap</li>
  </ul>
</App>
```

当然，事情肯定不会这么简单人都是善变的，现在和你说标题永远不变，转身就变给你看：如果用户没参加过，那就展示另一个灰色提示、如果用户分数很高，给他标题加一个角标、如果充钱，标题换个皮肤

我们还是保持尽量少改核心逻辑原则，先把header部分抽出一个组件：

```javascript
const [IS_NO_JOIN, IS_PAY, IS_SCORE_HIGH] = [1, 2, 3]
const MAP = {
  [IS_NO_JOIN]: 'no-join',
  [IS_PAY]: 'has-pay',
  [IS_SCORE_HIGH]: 'high-score',
}
function Header(props) {
  const [type, setType] = useState('normal')
  useEffect(() => {
    // 用名字获取用户身份
    requestTypeByName(props.children).then(res => {
      setType(res.type)
    })
  }, [])
  return (
    <header
      className={classnames(MAP[type])}
    >
      {props.children}
    </header>
  )
}
```

核心逻辑还是不动，渲染的结构和顺序也不动。更多定制化的功能，我们从上层的父组件控制，控制渲染逻辑、展示组件、显示的文案等等，上层的父组件拼装好，就通过props传入核心组件。这个过程就比如a同事之前写了核心渲染逻辑。**组件Header是a写的**，b同事不负责这块，但b是这次需求的开发者，所以应该把渲染的姿势调整好、适配，注入到a同事写的核心逻辑组件中去。最后的表现是，把控制权交给了b，让b来改，这样子也是侵入性小、风险小、扩展性强的一种方案。

> "成功甩了个锅"，a把文档甩给b，和b交代完逻辑后笑嘻嘻地下班了——这是比较优雅稳妥的方案，大家都理解的

反着来看，如果让a来改（假设核心模块是巨复杂而且无法快速入手的只能由a才能hold住的）：

```javascript
// 加样式n行
// 加处理state逻辑、加业务逻辑而且不能影响原有逻辑，改得小心翼翼
// 改字段来适配，又几行
  render() {
    const { name, score } = this.state.info
    return (
      <main>
        <header className="xxx">{name}</header>
        <section>{this.props.children}</section>
        分数：<section>{score}</section>
      </main>
    )
  }
// after coding： mmp，b的需求为什么要我出来帮忙，还要我从头开始看需求并了解需求
```

> 改完了，上线了，突然后面运营那边又说，要不就xxx....此时求a心里阴影面积

### 运营配置接口

前面的例子，我们都是做很小改动就完成了。当然每一次改个数字改个词，前端就要发布，这也不是优雅的解决方案。所以最终解决方案应该是，把这部分配置抽离出来，做到一个运营配置平台上，前端通过接口读取该平台的配置回来渲染页面。运营需要更改，我们只需要去平台上把配置修改即可。

```javascript
// 让配置写在一个可视化配置平台上
// const cgiMAp = {
//   '/a': {
//     name: 'nickname',
//     score: 'counts'
//   },
//   // ...
// }
function reqGenerator(cfg) {
  return Object.keys(cfg).reduce((res, key) => {
    res[key.slice(1)] = () => request(key).then(r => adapter(r, cfg[key]))
    return res
  }, {})
}
request('/config').then(res => {
  const reqs = reqGenerator(res.cgiMAp)
  reqs.a()
})
```

但是也要考虑一下缓存、容灾，提高一下页面的可用性、可访问性：

* 如果接口挂了，如何提升用户体验
* 怎样才能让页面掌控在手中，比如监控、埋点
* 怎样做到页面的健壮，经得起各种机器以及用户的乱操作的蹂躏

# 最后

放下前端这个标签，无论做的是什么，对自己都是一种成长，除了技术上，更多的是各种软技能、方法论、思维方式。出社会才发现，也许coding才是生活中最简单的一部分

> > 以上是工作一年以来的沉淀的一部分，算得上是工作总结吧，待续......

***若作者看到后，觉得很不爽，请联系pr@meishakeji.com。拜谢***