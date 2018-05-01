---
title: 如何配置Webpack系列一之conf.js
date: 2018-04-30 14:00:00
tags:
- Webpack
---

## 一、前言
> 
刚开始使用webpack的人在看webpack的配置的时候经常会头疼，这么多的配置文件，我应该如何快速对其进行自定义配置，并好奇其内在的联系又是怎么样的。
本系列将结合配置文件的代码来对webpack的配置进行阐述
限于篇幅，本系列主要分为两部分。第一部分，也就是本文，将通过三个webpack配置文件的代码来初步阐述应该如何配置webpack。

<!-- more -->

## 二、webpack.base.conf，webpack.base.dev，webpack.base.prod 之间的关系
> 这三个配置文件的关系很好理解，base也就是webpack基础配置部分，而dev配置和prod配置是在base的基础上，通过代码运行的环境来决定究竟使用dev配置还是prod配置。这里简单介绍一下dev 和 prod的含义。 dev即development，指的是开发环境，即我们常常对网页进行开发和调试所处的环境。 prod即production，指的是实际的生产环境。

## （1）webpack.base.conf.js
>这里webpack.base.conf.js主要用于：
1、配置webpack入口文件
2、配置输出路径
3、配置模块路径解析resolve的规则
4、配置对不同类型模块进行处理的规则
```js
'use strict'
// 这里引入了 一个npm模块（path） 和 三个配置文件（utils, config, vue-loader）
const path = require('path') //本模块包含一套用于处理和转换文件路径的工具集。
const utils = require('./utils')
const config = require('../config')
const vueLoaderConfig = require('./vue-loader.conf')

function resolve (dir) { //获取与当前文件夹同级的绝对路径
  return path.join(__dirname, '..', dir) // __dirname指当前文件所在的文件夹的绝对路径，在此的当前文件指webpack.base.conf.js
}

module.exports = {

  entry: { //entry以k-v形式配置webpack打包后的入口文件和打包前的入口文件（支持配置多入口，这里不作展开）
    app: './src/main.js'
  },

  output: { //webpack输出路径和命名规则
    path: config.build.assetsRoot, // webpack输出的目标文件夹路径（例如：/dist）
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production' // 待补充
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {// 模块resolve的规则
    extensions: ['.js', '.vue', '.json'],
    alias: { // 自定义别名。（后缀$表示精准匹配，可参考http://www.css88.com/doc/webpack2/configuration/resolve/）
      'vue$': 'vue/dist/vue.esm.js', // import Vue from 'vue/dist/vue.esm.js' 等于 import Vue from 'vue'
      '@': resolve('src'), // import router from '@/router' 等于 import router from './src/router';
    }
  },
  module: {
    rules: [
      {// 对src和test文件夹下的.js和.vue文件使用eslint-loader进行代码规范检查
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {// 对所有.vue文件使用vue-loader进行编译
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {// 对src和test文件夹下的.js文件使用babel-loader将es6+的代码转成es5
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {// 对图片资源文件使用url-loader
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,// 小于10K的图片转成base64编码的dataURL字符串写到代码中 (以下同理)
          name: utils.assetsPath('img/[name].[hash:7].[ext]')// 大于10K的图片转移到静态资源文件夹
        }
      },
      {// 对多媒体资源文件使用url-loader
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {// 对字体资源文件使用url-loader
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}
```
## （2）webpack.dev.conf.js
>这里webpack.dev.conf.js主要用于：
1、将webpack的热重载客户端代码添加到每个入口对应的应用
2、合并基础的webpack配置
3、配置样式文件的处理规则
4、配置Source Maps
5、配置webpack插件
```js
'use strict'
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')// webpack-merge是一个用于合并对象或数组并返回一个新的对象或数组的方法
const baseWebpackConfig = require('./webpack.base.conf') // webpack的基础配置文件
const HtmlWebpackPlugin = require('html-webpack-plugin') // 可自动在index.html中注入webpack打包后的文件
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin') // 该插件可以准确识别webpack的报错，提供更好的开发体验
// add hot-reload related code to entry chunks
Object.keys(baseWebpackConfig.entry).forEach(function (name) {// 给每个入口页面(应用)加上dev-client，用于跟dev-server的热重载插件通信，实现热更新
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})

module.exports = merge(baseWebpackConfig, {
  module: {
// 样式文件的处理规则，对css/sass/scss等不同内容使用相应的styleLoaders
// 由utils配置出各种类型的预处理语言所需要使用的loader，例如sass需要使用sass-loader
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  // cheap-module-eval-source-map is faster for development
  devtool: '#cheap-module-eval-source-map', //sourceMap是用于记录压缩的翻译文件，通过这个文件可以找到压缩文件中代码所对应的源码
  plugins: [ // 创建要用到的webpack的插件
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
    new webpack.HotModuleReplacementPlugin(), // 在页面不刷新的前提下实现修改、添加或删除前端页面中的模块代码。
    new webpack.NoEmitOnErrorsPlugin(), // 保证程序运行出错时页面不阻塞，且会在编译结束后报错。
    // https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({ // 注入webpack打包文件的html的创建。当webpack打包文件使用哈希值作为文件名，并且每次编译的哈希值都不同时尤其有用，防止引用缓存的外部文件问题
      filename: 'index.html', // 设置生成的 html 文件的文件名。
      template: 'index.html', // 根据自己的指定的模板文件来生成特定的 html 文件。
      inject: true // 注入选项。有四个选项值 true(默认值，script标签位于html文件的 body 底部), body(同true), head（script 标签位于 head 标签内）, false（不插入生成的 js 文件，只是单纯的生成一个 html 文件）.
    }),
    new FriendlyErrorsPlugin()
  ]
})
```



## （3）webpack.prod.conf.js
>这里webpack.prod.conf.js主要用于：
1、合并基础的webpack配置
2、配置样式文件的处理规则，styleLoaders
3、配置webpack的输出
4、配置webpack插件
5、gzip模式下的webpack插件配置
6、构建包webpack-bundle分析
```js
'use strict'
const path = require('path')
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
const CopyWebpackPlugin = require('copy-webpack-plugin') // 在webpack中拷贝文件和文件夹
const HtmlWebpackPlugin = require('html-webpack-plugin') // （参考dev配置）
const ExtractTextPlugin = require('extract-text-webpack-plugin') // 主要是为了抽离css样式,防止将样式打包在js中引起页面样式加载错乱的现象
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin') // 压缩提取出的css，并解决ExtractTextPlugin分离出的js重复问题(多个文件引入同一css文件)

const env = config.build.env

const webpackConfig = merge(baseWebpackConfig, {
  module: {
    // （参考dev配置）
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  devtool: config.build.productionSourceMap ? '#source-map' : false, // 是否使用source-map
  output: { // webpack输出路径和命名规则
    path: config.build.assetsRoot, // 通常为'/'
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': env
    }),
    // UglifyJs do not support ES6+, you can also use babel-minify for better treeshaking: https://github.com/babel/minify
    new webpack.optimize.UglifyJsPlugin({ // 丑化压缩JS代码
      compress: {
        warnings: false
      },
      sourceMap: true,
      parallel: true
    }),
    // extract css into its own file
    new ExtractTextPlugin({ // 抽离css样式
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    new OptimizeCSSPlugin({ // 压缩提取出的css，如果只简单使用extract-text-plugin可能会造成css重复
      cssProcessorOptions: {
        safe: true
      }
    }),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: config.build.index,
      template: 'index.html',
      inject: true,
      minify: { // 简化index.html
        removeComments: true, // 删除注释
        collapseWhitespace: true, // 删除空格
        removeAttributeQuotes: true // 删除属性值的双引号
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency' // 注入依赖的时候按照依赖先后顺序进行注入，比如，需要先注入vendor.js，再注入app.js
    }),
    // keep module.id stable when vender modules does not change
    new webpack.HashedModuleIdsPlugin(), // 将所有从node_modules中引入的js以哈希文件名的形式提取到vendor.js，即抽取库文件，
    // split vendor js into its own file
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    new webpack.optimize.CommonsChunkPlugin({ // 从vendor中提取出manifest，防止每当app包更新的时候，vendor的哈希值也跟着更新
      name: 'manifest',
      chunks: ['vendor']
    }),
    // copy custom static assets
    new CopyWebpackPlugin([ // 将static里的资源复制到一个自定义的路径，此处为config.build.assetsSubDirectory
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})

if (config.build.productionGzip) { // 能够将资源文件压缩为.gz文件,并且根据客户端的需求按需加载
  const CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

if (config.build.bundleAnalyzerReport) { // 可以将打包后的内容树展示为方便交互的直观树状图，直观看到所构建包中真正引入的内容
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig
```