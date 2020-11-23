---
title: @vue/cli-service 源码分析
date: 2020-11-17 08:30:00
tags:
- vue CLI
- @vue/cli-service
---

使用 @vue/cli 创建项目的开发小伙伴们，每天打开项目代码，开始日常的开发前，都会在项目的根目录运行 `npm run xxx` ，xxx 是我们在 package.json文件的 `script` 字段中定义的开启本地服务的命令名。从我们输入完命令，按下enter 键到启动服务成功 @vue/cli 为我们做了些什么呢？今天我们就一起来探究探究吧。

使用 @vue/cli 创建的项目都会局部安装上 @vue/cli-service 包，我们能够零配置快速启动、打包项目，都是它的功能，得助于 @vue/cli-service 内部添加的 webpack 配置。

为了能够从原理上了解 @vue/cli-service 的工作流程，我们就一起来从它的源码来分析，在本地运行项目时，@vue/cli-service 做了些什么。


### 背景知识

1. @vue/cli-service 内部已经编写了必要的 webpack 配置，能够对静态资源，CSS 相关等依赖进行打包， 这些配置是通过  [webpack-chain](https://github.com/neutrinojs/webpack-chain) 编写的。

2. @vue/cli-service npm 包的文件结构
![image.png](https://cdn.nlark.com/yuque/0/2020/png/1579161/1605569055603-a3803701-a88b-42a7-80f3-67361a46f2a2.png)

3. 内部提供给用户添加自定义配置的三种方式：

   在项目根文件目录，添加 vue.config.js 文件；

   在 package.json 文件中的vue字段；

   初始化 @vue/cli-service 的 Server时，

以上三种方式的优先级依次为：vue.config.js > package.json > inline, 重复使用不同的方式，低优先级的配置会被忽略。

4. 添加自定义配置，主要在 `configureWebpack` 或者 `chainWebpack`添加 webpack 相关的配置。

- configureWebpack ：如果这个值是一个对象，则会通过 [webpack-merge](https://github.com/survivejs/webpack-merge) 合并到最终的配置中。如果这个值是一个函数，则会接收被解析的配置作为参数。该函数既可以修改配置并不返回任何东西，也可以返回一个被克隆或合并过的配置版本。
- chainWebpack ：接受一个基于 [webpack-chain](https://github.com/mozilla-neutrino/webpack-chain) 的 ChainableConfig 实例。允许对内部的 webpack 配置进行更细粒度的修改 configureWebpack


### 入口

因为需要通过 `vue-cli-service` 命令启动项目，所以我们就从仓库的package.json 的 `bin` 字段入手，从 `bin` 得知，运行 vue-cli-service 命令会执行仓库的 bin/vue-cli-service.js 文件。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1579161/1605511590858-5cee2876-93b5-4ae1-91f3-5cb21b6400a6.png)

```
// ...
const Service = require('../lib/Service')
const service = new Service(process.env.VUE_CLI_CONTEXT || process.cwd())

const rawArgv = process.argv.slice(2)
const args = require('minimist')(rawArgv, {
  boolean: [
    // build
    'modern',
    'report',
    'report-json',
    'watch',
    // serve
    'open',
    'copy',
    'https',
    // inspect
    'verbose'
  ]
})
const command = args._[0]

service.run(command, args, rawArgv).catch(err => {
  error(err)
  process.exit(1)
})
```

在 bin/vue-cli-service.js 里做了以下工作:

   引用了 '../lib/Service'，并实例化了 Service；

   获取命令行参数；

   执行 Service实例 的 run 方法。



接下来我们就来看看创建 Servcie 经历了什么，以及 run 方法里又发生了什么事情。



### 创建 Service 实例

```
class Service {
  constructor (context, { plugins, pkg, inlineOptions, useBuiltIn } = {}) {
    process.VUE_CLI_SERVICE = this
    this.initialized = false
    this.context = context
    this.inlineOptions = inlineOptions
    this.webpackChainFns = []
    this.webpackRawConfigFns = []
    this.devServerConfigFns = []
    this.commands = {}
    this.pkgContext = context
    this.pkg = this.resolvePkg(pkg)
    this.plugins = this.resolvePlugins(plugins, useBuiltIn)
    this.modes = this.plugins.reduce((modes, { apply: { defaultModes }}) => {
      return Object.assign(modes, defaultModes)
    }, {})
  }
  // ....
}
```

在 Service 的构造函数中，主要添加了一些属性：

- initialized: 一个标示，在后续会用到，用于控制每个 Service 只初始化一次(执行一次 init 方法)。
- context：运行 @vue/cli-service 的文件路径
- inlineOptions: 在调用 Service 时手动传入的项目配置
- webpackChainFns:  chainWebpack 添加的配置
- webpackRawConfigFns: configureWebpack 添加的配置
- devServerConfigFns: webpack-dev-serve 的devServer 配置
- commands: Service 所有可用的命令，会在后续注册
- pkgContext: 项目package.json 的绝对文件路径
- pkg: 一个解析的 package.json 数据
- plugins：内置的和第三方的 CLI 插件，resolvePlugins 返回的是数组，数组每一项为包含插件id和插件方法的对象。
- modes: 运行模式，serve，build，inspect分别对应'development', 'production'， 'development'

内置插件有

```
[
  './commands/serve', './commands/build', './commands/inspect', './commands/help',
  // config plugins are order sensitive
  './config/base', './config/css', './config/dev', './config/prod',  './config/app'
]
```

如果你对其中一些属性感到迷惑，不用担心，我们会在后续的介绍到。


### run 方法

在实例化 Service 之后，就调用了 `run` 方法

```
async run (name, args = {}, rawArgv = []) {
  const mode = args.mode || (name === 'build' && args.watch ? 'development' : this.modes[name])
  this.init(mode)
  let command = this.commands[name]
  // ...

  if (!command || args.help) {
    command = this.commands.help
  } else {
    args._.shift() // remove command itself
    rawArgv.shift()
  }
  const { fn } = command
  return fn(args, rawArgv)
}
```

在 run 方法中，执行了 init 方法

```
init (mode = process.env.VUE_CLI_MODE) {
  if (this.initialized) {
  	return
  }
  this.initialized = true
  this.mode = mode

  // 加载环境变量，例如 .env.dev, .env.dev.local
  if (mode) {
    this.loadEnv(mode)
  }
  this.loadEnv()

  const userOptions = this.loadUserOptions() // 获取用户自定义配置
  this.projectOptions = defaultsDeep(userOptions, defaults())

  this.plugins.forEach(({ id, apply }) => {
    apply(new PluginAPI(id, this), this.projectOptions)
  })

  // apply webpack configs from project config file
  if (this.projectOptions.chainWebpack) {
    this.webpackChainFns.push(this.projectOptions.chainWebpack)
  }
  if (this.projectOptions.configureWebpack) {
    this.webpackRawConfigFns.push(this.projectOptions.configureWebpack)
  }
}
```

init 方法中：

1. 加载环境变量；
2. 从 vue.config.js 、package.json 的 vue 字段、inlineOptions，获取用户自定义配置，优先级递减；
3. 合并 cli 的默认配置和用户自定义的 @vue/cli-service 配置；
4. PluginAPI 接受插件的 id 和 Service 实例为参数，创建一个可以读写 Service 实例的属性和方法的实例；
5. 插件接受 PluginAPI 实例和 cli 配置两个参数；
6. 添加用户定义的 `configureWebpack`，`chainWebpack` 字段。

经过遍历 plugins，调用插件方法，注册了所有 @vue/cli-service 命令：`serve`，`build`，`lint`，`inspect`。组合了默认的 webpack 配置。

`./commands/*` 里都会调用 `PluginAPI` 的 `registerCommand` 方法，以此来注册 @vue/cli-service 命令

```
// ./commands/*
(api, options) => {
  api.registerCommand(/* */);
}
// PluginAPI
registerCommand (name, opts, fn) {
  if (typeof opts === 'function') {
    fn = opts
    opts = null
  }
  this.service.commands[name] = { fn, opts: opts || {}}
}
```

`./config/*`  里会调用 PluginAPI 的 chainWebpack 方法, 这里就是添加默认的打包配置，如果你想知道你的项目经过怎样的打包处理

就可以看看这些文件里添加了哪些配置。在添加配置中使用的是 `webpack-chain`的语法，如果你想更加直观的查看配置，也可以运行 @vue/cli-service 的 inspect 命令。它会返回一个最终的 webpack 配置对象

```
// ./config/*
(api, options) => {
  api.chainWebpack(）
}
// PluginAPI
chainWebpack (fn) {
  this.service.webpackChainFns.push(fn)
}
```

经过这几个步骤之后，init方法就执行完成，然后再回到 run 方法中，在run 方法中，最后就执行了命令对应的方法。


从输入命令按下enter 到最终执行命令的整个过程就到此结束了，现在我们来总结一下整个流程：

获取环境变量 ---> 获取用户的 service-cli 配置 --->注册 service-cli 命令 ---> 生成内置的 webpack 配置 ----> 将用户的 service-cli 配置中的webpack 配置添加 Service 中 ----> 执行输入的 service-cli 命令。


本次探索之旅到此结束了，感谢大家的阅读！



