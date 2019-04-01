---
title: 使用meishaFE自定义mpvue脚手架开发微信小程序
date: 2019-02-20 17:00
tags:
- mpvue
- 微信小程序
---

## 写在开头
mpvue [github地址](https://github.com/Meituan-Dianping/mpvue)  是一套定位于开发小程序的前端开发框架，其核心目标是提高开发效率，增强开发体验。使用该框架，开发者只需初步了解小程序开发规范、熟悉Vue.js基本语法即可上手。

##用公司自定义脚手架来开发你的项目
框架提供了完整的 Vue.js 开发体验，开发者编写Vue.js代码，mpvue 将其解析转换为小程序并确保其正确运行。此外，框架还通过 vue-cli 工具向开发者提供quick start 示例代码，开发者只需执行一条简单命令，即可获得可运行的项目。

当然，也可以使用我们公司的自定义脚手架 [github地址](https://github.com/meishaFE/mpvue-quickstart) 来搭建自己的项目：
<!-- more -->
```rst
$ npm install -g vue-cli
$ vue init meishaFE/mpvue-quickstart my-project
$ cd my-project
$ npm install
$ npm run dev
```
运行后的项目目录如下：
```rst
.
├── README.md
├── build
│   ├── build.js
│   ├── check-versions.js
│   ├── dev-client.js
│   ├── dev-server.js
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
├── config
│   ├── dev.env.js
│   ├── index.js
│   └── prod.env.js
├── index.html
├── package.json
├── project.config.json
├── src
│   ├── App.vue
│   ├── app.json
│   ├── components
│   │   └── card.vue
│   ├── config
│   │   └── api.js
│   ├── main.js
│   ├── pages
│   │   └── counter
│   │       └── index.vue
│   ├── pages.js
│   ├── store
│   │   ├── actions.js
│   │   ├── getters.js
│   │   ├── index.js
│   │   └── mutations.js
│   └── utils
│       ├── fetch.js
│       └── index.js
└── static
```
## 自定义脚手架新增配套

- 请求拦截器

```javascript
const Fly = require('flyio/dist/npm/wx');

export const fly = new Fly();

// 设置超时
fly.config.timeout = 0;
// 设置请求基地址
fly.config.baseURL = BASE_URL;

// 添加请求拦截器
fly
  .interceptors
  .request
  .use((request) => {
   // **
  });

// 添加响应拦截器，响应拦截器会在then/catch处理之前执行
fly
  .interceptors
  .response
  .use(res => {
    // **
  }, err => {
    // 发生网络错误后会走到这里
    return Promise.reject(new Error(`Request failed with status code ${err.status}`));
  });
```

- 开发环境、测试环境、生产环境切换：

```javascript
// 当前环境
const CURRENT_ENV = PROD;

// 是否为开发环境
export const IS_DEV = CURRENT_ENV === DEV;

// 是否为测试环境
export const IS_TEST = CURRENT_ENV === TEST;

// 是否为预发布环境
export const IS_PRE = CURRENT_ENV === PRE;

// 是否为正式环境
export const IS_PROD = CURRENT_ENV === PROD;

// 环境
export const ENV = {
  IS_DEV,
  IS_TEST,
  IS_PRE,
  IS_PROD
};

// 各个环境的 URL 列表
const URL_SET = {
  [DEV]: {
    API_BASE_URL: 'https://vito-evaluate2.meishakeji.com'
  },
  [TEST]: {
    API_BASE_URL: 'https://test-evaluate2.meishakeji.com'
  },
  [PRE]: {
    API_BASE_URL: 'https://pre-evaluate2.meishakeji.com'
  },
  [PROD]: {
    API_BASE_URL: 'https://evaluate2.meishakeji.com'
  }
};
```

- MpvueEntry, 将小程序的index.json放在一个文件下配置管理

```javascript
/** webpack.base.conf.js **/

const MpvueEntry = require('mpvue-entry');
module.exports = {
  entry: MpvueEntry.getEntry('src/pages.js')
}

/*** pages.js ***/

module.exports = [
  // {
  //   path: 'pages/news/list', // 页面路径，同时是 vue 文件相对于 src 的路径，必填
  //   config: { // 页面配置，即 page.json 的内容，可选
  //     navigationBarTitleText: '标题',
  //     enablePullDownRefresh: true
  //   }
  // }
  {
    path: 'pages/counter/index',
    config: {
      navigationBarTitleText: '计数器'
    }
  }
];
```
## mpvue开发小程序需要注意的地方
####1. css单位：
mpvue有px2rpx-loader 样式转化插件配套设施：
```javascript
var px2rpxLoader = {
    loader: 'px2rpx-loader',
    options: {
      baseDpr: 1,
      rpxUnit: 0.5
    }
  }
```
因此写样式只用写375px宽度下的px值就能转化为wxss下对应的rpx：
```sass
.home {
  display: inline-block;
  margin: 100px auto;
  padding: 5px 10px;
  color: blue;
  border: 1px solid blue;
}
```

####2. mpvue框架下的小程序的生命周期钩子onShow和mounted
onshow：页面一旦展示就会触发
mounted: 页面进入小程序页面栈时候触发
![](https://lexiangla.com/assets/45e6a54c34ef11e9aea3525400b4d70f)

####3. textarea层级无法覆盖问题，解决办法
![](https://lexiangla.com/assets/83689b301a3e11e9b1f45254004f9daa)
![](https://lexiangla.com/assets/8ded4de41a3e11e99ded525400b4d70f)
上面两处的button组是固定在页面底部的，然后当页面进来时，这个地址和兴趣爱好正好在button的下面。最后在修改其他信息保存时候，点击保存，会发现调起了键盘。
原因是textarea组件在小程序页面中的层级是最高的，定位/index什么的都无法盖住。真是绝望。。。
后来发现小程序的coverview组件可以覆盖住，但是限制是coverview里面只能放coverview标签，所以button 按钮的样式又要重新写了。所以，总结一下，这里由于小程序的种种限制，建议在过设计稿时候提前告知，能改设计最好，毕竟这种操作还是有很多局限性。

####4. 登录流程图
划重点： 首先就是登录模块一定要和后台商量好，这里的逻辑不少。要考虑到直接进到某个页面的情况。
登录这块最开始用的是教务的那套登录流程，后来听了廖神的话，把这块逻辑改了，重新做了一套登录流程，花了时间不说，最后又发现不太合理。又得改回来，导致登录这块的逻辑一直变。最后自己还是根据最后的登录逻辑，做了一个流程图，希望能帮助到大家。
![](https://lexiangla.com/assets/fd46cc841a3d11e9bb90525400a20cd4)

## 最后
磨刀不误砍柴工，熟读mpvue的开发文档(例如：生命周期钩子的理解)，会让后面开发少走好多弯路。mpvue不建议使用的组件(例如：slot)能不用就不用。希望大家的mpvue用起来能更顺利。