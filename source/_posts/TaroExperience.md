---
title: Taro初体验（一）
date: 2018-07-04 09:48:38
author: aking
tags:
- JavaScript
---


## 前言

最近想了解一下小程序的开发，刚好看到京东的 o2team 开源了一套遵循 React 语法规范的框架，因为之前对 React 比较熟悉，所以赶快上手体验了一下。

## Taro 简介

👽 Taro，泰罗·奥特曼，宇宙警备队总教官，实力最强的奥特曼。

Taro 的语法规则基于 React 规范，有着与 React 一致的组件化思想，生命周期也和 React 生命周期一致不需要学习小程序的那一套生命周期，而且支持使用 JSX 语法！这点就是选择 Taro 很重要的理由了，我觉得JSX语法更加语义化且写起来可以让代码更加简洁干净。总之就是只写一份 React 语法的代码，可以用 Taro 转换成多个端运行的代码（小程序、H5、ReactNative，快应用）🤩。

## Taro 的使用

安装 Taro 就多不赘述，可以去 [github地址](https://github.com/NervJS/taro) 查看，推荐安装用 cnmp ，之前用 yarn 和 npm 都报了错。

安装后文件目录如下：

```
├── dist                   编译结果目录
├── config                 配置目录
|   ├── dev.js             开发时配置
|   ├── index.js           默认配置
|   └── prod.js            打包时配置
├── src                    源码目录
|   ├── pages              页面文件目录
|   |   ├── index          index页面目录
|   |   |   ├── index.js   index页面逻辑
|   |   |   └── index.css  index页面样式
|   ├── app.css            项目总通用样式
|   └── app.js             项目入口文件
└── package.json
```

 如果引入了redux，例如我们的项目，目录是这样的

```
├── dist                   编译结果目录
├── config                 配置目录
|   ├── dev.js             开发时配置
|   ├── index.js           默认配置
|   └── prod.js            打包时配置
├── src                    源码目录
|   ├── actions            redux里的actions
|   ├── asset              图片等静态资源
|   ├── components         组件文件目录
|   ├── constants          存放常量的地方，例如api、一些配置项
|   ├── reducers           redux里的reducers
|   ├── store              redux里的store
|   ├── utils              存放工具类函数
|   ├── pages              页面文件目录
|   |   ├── index          index页面目录
|   |   |   ├── index.js   index页面逻辑
|   |   |   └── index.css  index页面样式
|   ├── app.css            项目总通用样式
|   └── app.js             项目入口文件
└── package.json
```

app.js 入口文件示例如下：

```javascript
import Taro, { Component } from '@tarojs/taro'
import Index from './pages/index'
import './app.scss'

class App extends Component {
  config = {
    pages: [
      'pages/index/index',
      'pages/logs/logs',
      'pages/login/login',
      'pages/lesson/lesson'
    ],
    window: {
      backgroundTextStyle: 'light',
      navigationBarBackgroundColor: '#eb941e',
      navigationBarTitleText: '焙刻',
      navigationBarTextStyle: 'white'
    },
    tabBar: {  
      color: "#a9b7b7",  
      selectedColor: "#eb941e",  
      borderStyle: '#fff',
      list:[
        {
          pagePath:"pages/index/index",
          selectedIconPath:"./image/cook_icon.png",
          iconPath:"./image/cook_icon1.png",
          text: "烘焙圈"
        },
        {
          pagePath:"pages/logs/logs",
          selectedIconPath:"./image/cake.png",    
          iconPath:"./image/cake1.png",          
          text: "味空间"
        },
        {
          pagePath:"pages/lesson/lesson",
          selectedIconPath:"./image/silverware-variant.png",          
          iconPath:"./image/silverware-variant1.png",          
          text: "烘焙课"
        },
        {
          pagePath:"pages/login/login",
          selectedIconPath:"./image/House.png",
          iconPath:"./image/House1.png",                
          text: "我的"
        }
      ]
    },  
  }
  componentDidMount () {}

  componentDidShow () {}

  componentDidHide () {}

  componentCatchError () {}
  render () {
    return (
      <Index />
    )
  }
}
Taro.render(<App />, document.getElementById('app'))
```

可以看出入口文件也是 React 风格的写法，首先要引用依赖 @tarojs/taro，在这里我们继承了 Component 类。

通常入口文件会包含一个 config 配置项，这里的配置主要参考微信小程序的[全局配置](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html)而来，在编译成小程序时，这部分配置将会被抽离成 app.json，如果编译成其他端(H5)，也会有其他用途（待深入了解）。

创建 Taro 项目时会提示选择 sass 还是 less，这里我们选择了 sass，在 app.scss 定义全局的样式，其定义的样式会作用于每个页面，当然页面本身的局部样式也可以覆盖全局样式。

接下来我们尝试写一个小程序 DEMO 主页：

``` js
import Taro, { Component } from '@tarojs/taro'
import { View, Text, Swiper, SwiperItem, Image } from '@tarojs/components'
import whatshot from '../../image/whatshot.png'
import './index.scss'
export default class Index extends Component {
  constructor () {
    super(...arguments)
    this.state = {}
    }
  config = {
    navigationBarTitleText: '焙刻'
  }
  componentWillMount() {}
  componentDidMount() {
    Taro.showLoading({title: '加载中'})
      setTimeout(()=>{
          Taro.hideLoading()
      },1000)
  }
  componentWillUpdate () {}
  
  componentDidUpdate () {}
 
  shouldComponentUpdate () {}

  render() {
    return (
      <View className='index'>
        <Swiper autoplay
          indicatorDots
          circular
          interval={2000}
          style='height:220px;'
          indicatorActiveColor='#fff'
        >
          {imageList.map((item,index)=>{
               return (
                <SwiperItem key={index}>
                    <Image src={item.imgUrl} />
                </SwiperItem>
               )
            })}
            </Swiper>
            <View className='index__iconGroup'>
                <View className='index__iconRow'>                
                    {
                      iconList1.map((item,index)=>{
                        return (
                        <View className='index__iconContainer' key={index}>
                          <Image src={item.iconUrl} />
                          <Text>{item.text}</Text>
                        </View>
                        )
                        })
                    }
                </View>
            </View>
            <View className='index__content'>
                <View className='index__title'>
                <Image src={whatshot}></Image>
                    热门话题
                </View>
                <View style='display:flex;justify-content:space-between;'>
                    <View className='index__card'>         
                        <View className='index__cardPic'>
                            <Text>玩烘焙</Text>
                            <Text>轻松搞定烘焙技能</Text>
                        </View>
                    </View>
                    <View className='index__card index__cardImg'>         
                        <View className='index__cardPic'>
                            <Text>晒分享</Text>
                            <Text>分享讨论你的幸福</Text>
                        </View>
                    </View>
                </View>
                <View>
                    <View className='index__cardPizza'>
                            <View className='index__cardPic'>
                                <Text>一入烘焙深似海</Text>
                            </View>
                        </View>
                    </View>
                </View>
      </View>
    )
  }
}
```

效果图：

![bake](http://ovwvaynot.bkt.clouddn.com/bake.gif)

Taro 的数据请求 api 使用了 Taro.request， 跟 wx.request 使用方法基本一致，不同的是 Taro.request 天然支持 promise 化，点击跳转使用 Taro.navigateTo，跟 wx.navigateTo 使用方法基本一致。

Taro 引用本地静态资源需要先 import 进来再使用，为了让 h5 部署的时候图片路径不出错，最好把图片放到七牛上，然后写路径。

```js
// Taro 引用本地静态资源
import icon from '../../images/icon.png'
<Image src={icon}></Image>

// 最好把图片放在服务器上，然后写http 路径
<Image src='https://123.icon.png'></Image>
```

## 总结

原生开发和 Taro 开发小程序，Taro 可以增加代码复用性，很适合需要 web 端、小程序端等多端支持的项目，尤其是已有 react 实现的 web 产品，需要快速开发一个小程序版本的需求场景，并且该团队更熟悉的是 react。

Taro 最主要的功能是 web 端与小程序一致的开发体验，以及尽可能的实现了 UI 复用，比如一些 toast、showLoading 等。

总的来说，Taro 作为一个新的形态，从开发角度来看，它可以算是多端的一个结合，开发一套代码既能生成快应用，也能生成其他端，节约开发成本、开发时间等。Taro 支持组件化开发、npm 包支持，css 预编译器支持等等，最重要是可以用 JSX 来写微信小程序，react 粉可以快速上手了。再细节的许多点还需要深入体验一下，希望大家也能多多尝试。

 

 

 

 

 