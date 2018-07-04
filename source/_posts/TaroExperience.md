---
title: Taro初体验（一）
date: 2018-07-04 09:48:38
author: aking
tags:
- JavaScript
---


## 前言

最近想了解一下小程序的开发，刚好看到京东的凹凸实验室开源了一套遵循React语法规范的框架，因为直接对React比较熟悉，所以赶快上手体验了一下。

## Taro简介

👽 Taro，泰罗·奥特曼，宇宙警备队总教官，实力最强的奥特曼。

Taro的语法规则基于React规范，有着与React一致的组件化思想，生命周期也和React生命周期一致不需要学习小程序的那一套生命周期，而且支持使用JSX语法！这点就是选择Taro很重要的理由了，我觉得JSX语法更加语义化且写起来可以让代码更加简洁干净。总之就是只写一份React语法的代码，可以用Taro转换成多个端运行的代码（小程序、H5、ReactNative）🤩。

## Taro的使用

安装Taro就多不赘述，可以去 [github地址](https://github.com/NervJS/taro)查看，推荐安装用cnmp，之前用yarn和npm都报了错。

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

app.js入口文件示例如下：

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

可以看出入口文件也是React风格的写法，首先要引用依赖 @tarojs/taro，在这里我们继承了Component类。

通常入口文件会包含一个config 配置项，这里的配置主要参考微信小程序的[全局配置](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html)而来，在编译成小程序时，这部分配置将会被抽离成app.json，如果编译成其他端（H5），也会有其他用途（待深入了解）。

创建Taro项目时会提示选择sass还是less，这里我们选择了sass，在app.scss定义全局的样式，其定义的样式会作用于每个页面，当然页面本身的局部样式也可以覆盖全局样式。

接下来我们尝试写一个小程序DEMO主页：

``` js
import Taro, { Component } from '@tarojs/taro'
import { View, Text, Swiper,SwiperItem,Image} from '@tarojs/components'
import whatshot from '../../image/whatshot.png'
import './index.scss'
export default class Index extends Component {
  constructor () {
    super(...arguments)
    this.state = {
    }
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
 
  shouldComponentUpdate () {
    return true
  }
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
                    <Image src={item.imgUrl} style='width:100%;height:100%;' />
                </SwiperItem>
               )
            })}
            </Swiper>
            <View className='index__iconGroup'>
                <View className='index__iconRow'>                
                    {iconList1.map((item,index)=>{
                        return (
                        <View className='index__iconContainer' key={index}>
                          <Image src={item.iconUrl} style='width:40px;height:40px;' />
                          <Text>{item.text}</Text>
                        </View>
                        )
                        })}
                </View>
            </View>
            <View className='index__content'>
                <View className='index__title'>
                <Image src={whatshot} style='width:44rpx;height:44rpx;'></Image>
                    热门话题
                </View>
                <View style='display:flex;justify-content:space-between;'>
                    <View className='index__card'>         
                        <View className='index__cardPic'>
                            <Text style='font-size:1.2rem;'>玩烘焙</Text>
                            <Text>轻松搞定烘焙技能</Text>
                        </View>
                    </View>
                    <View className='index__card index__cardImg'>         
                        <View className='index__cardPic'>
                            <Text style='font-size:1.2rem;'>晒分享</Text>
                            <Text>分享讨论你的幸福</Text>
                        </View>
                    </View>
                </View>
                <View>
                    <View className='index__cardPizza'>
                            <View className='index__cardPic'>
                                <Text style='font-size:1.2rem;'>一入烘焙深似海</Text>
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

![bake](/Users/aking/Desktop/bake.gif)

Taro的数据请求api使用了Taro.request， 跟wx.request 使用方法基本一致，不同的是 Taro.request 天然支持 promise 化，点击跳转使用Taro.navigateTo，跟wx.navigateTo使用方法基本一致。

Taro引用本地静态资源需要先import进来再使用，为了让h5部署的时候图片路径不出错，最好把图片放到七牛上，然后写路径。

```js
// Taro 引用本地静态资源
import icon from '../../images/icon.png'
<Image src={icon}></Image>

// 最好把图片放在服务器上，然后写http 路径
<Image src='https://123.icon.png'></Image>
```

## 总结

原生开发和taro开发小程序，taro可以增加代码复用性，很适合需要web端、小程序端等多端支持的项目，尤其是已有react实现的web产品，需要快速开发一个小程序版本的需求场景，并且该团队更熟悉的是react。

taro最主要的功能是web端与小程序一致的开发体验，以及尽可能的实现了ui复用，比如一些toast、showLoading等。

总的来说，taro作为一个新的形态，从开发角度来看，它可以算是多端的一个结合，开发一套代码既能生成快应用，也能生成其他端，节约开发成本、开发时间等。taro支持组件化开发、npm包支持，css预编译器支持等等，最重要是可以用jsx来写微信小程序，react粉可以快速上手了。再细节的许多点还需要深入体验一下，希望大家也能多多尝试。

 

 

 

 

 