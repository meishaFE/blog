---
title: egg项目初始化及其启动的分析
date: 2019-03-10 20:00:00
tags: 
- egg.js
- node
- Koa
---

## 前言
egg.js是由阿里出品的继承于Koa的应用框架。正如egg的介绍，egg没有选择社区常见框架的大集市模式（集成如数据库、模板引擎、前端框架等功能），而是专注于提供 Web 开发的核心功能和一套灵活可扩展的插件机制。
egg有以下特点：

<!-- more -->
>1.具有很高拓展性，具体体现在egg的插件机制，即一个插件只做一件事，如： 将Nunjucks模板封装成了egg-view-nunjucks，MySQL数据库封装成了 egg-mysql，结合实际业务场景聚合所需要的插件，这样应用的开发成本就变得很低
>2.Egg奉行约定优于配置，使用一套统一的约定进行应用开发，可在一套框架的配置的基础上，进行自定义配置：
```json
// 框架配置
// package.json
{
  "name": "framework1",
  "version": "1.0.0",
  "dependencies": {
    "egg-mysql": "^3.0.0",
    "egg-view-nunjucks": "^2.0.0"
  }
}
```
```js
// config/plugin.js
module.exports = {
  mysql: {
    enable: false,
    package: 'egg-mysql',
  },
  view: {
    enable: false,
    package: 'egg-view-nunjucks',
  }
}
```

此外，使用加载器（Loader）可以让框架根据不同环境定义默认配置，还可以覆盖 Egg 的默认约定
```json
// 应用配置
// package.json
{
  "dependencies": {
    "framework1": "^1.0.0",
  }
}
```
```js
// config/plugin.js
module.exports = {
  // 开启插件
  mysql: true,
  view: true,
}
```

那么，egg项目是怎么运行的呢，接下来结合源码做简单的分析：
我们用egg-init初始化一个egg项目出来，可以看到，项目是通过命令
egg-bin dev启动，直接进入到了/egg-bin/bin/egg-bin.js

```js
// egg-bin/bin/egg-bin.js
const Command = require('..');
new Command().start();
```
可以看到，egg-bin.js部分的代码比较简单，主要功能是引用了其上一层级的index.js，Command得到的值是Command的子类EggBin，挂载着CovCommand、DevCommand、TestCommand、DebugCommand和PkgfilesCommand这几个命令。接着实例化EggBin并执行其start方法启动项目，但在执行start之前，首先会执行EggBin的构造函数：

```js
// egg-bin/index.js
class EggBin extends Command {
  constructor(rawArgv) {
    // 获取用户输入的option
    super(rawArgv);
    ....
    // load对应目录下面的command文件
    this.load(path.join(__dirname, 'lib/cmd'));
  }
}
```
由于EggBin继承Command，Command又是继承common-bin的，所以我们直接找到common-bin：

```js
// common-bin/lib/command.js
this.rawArgv = rawArgv || process.argv.slice(2); // 获取进程中的参数，如：[0:"dev", 1:"--port", 2:"7001"]


// 加载存放Command命令的文件夹里所有命令文件的绝对路径，后续在EggBin上挂载一个键值对，键为命令的缩写，相应的值为这些绝对路径
load(fullPath) {
    ....
    // load entire directory
    const files = fs.readdirSync(fullPath);
    const names = [];
    for (const file of files) {
        if (path.extname(file) === '.js') {
            const name = path.basename(file).replace(/\.js$/, '');
            names.push(name); // [ 'autod', 'cov', 'debug', 'dev', 'pkgfiles', 'test' ]
            this.add(name, path.join(fullPath, file));
        }
    }
    ....
}

add(name, target) {
    ....
    if (!(target.prototype instanceof CommonBin)) {
        target = require(target);
    }
    ....
    this[COMMANDS].set(name, target);
}
```
可以看到这里执行了common-bin/command.js定义的load的方法，load里又调用了add将egg-bin/lib/cmd/文件夹里的所有用于执行命令行的文件以函数的形式加载到egg-bin实例的COMMANDS字段中，如下：
```js
// this[COMMANDS] = Map {
//   'autod' => [Function: AutodCommand],
//   'cov' => [Function: CovCommand],
//   'debug' => [Function: DebugCommand],
//   'dev' => [Function: DevCommand],
//   'pkgfiles' => [Function: PkgfilesCommand],
//   'test' => [Function: TestCommand] }
```

接着就是start的操作

```js
  start() {
    co(function* () {
      const index = this.rawArgv.indexOf('--get-yargs-completions');
      if (index !== -1) {
        this.rawArgv.splice(index, 2, `--AUTO_COMPLETIONS=${this.rawArgv.join(',')}`);
      }
      yield this[DISPATCH]();
    }.bind(this)).catch(this.errorHandler.bind(this));
  }

```
可以看到，start函数里面是一个自动执行的generator函数，其中执行了this[DISPATCH]，this[DISPATCH]函数如下：

```js

* [DISPATCH]() {
    ....
    const parsed = yield this[PARSE](this.rawArgv);

// parsed对传入的rawArgv进行处理，输出：
// { _: [ 'dev' ],
//   typescript: undefined,
//   ts: undefined,
//   declarations: undefined,
//   dts: undefined,
//   '$0':
//    '/Users/user/Documents/fedev/testegg/node_modules/.bin/egg-bin' }

    const commandName = parsed._[0];
    // 判断是否存在该子命令， 若执行npm run dev，那么commandName则为dev
    if (this[COMMANDS].has(commandName)) {
      const Command = this[COMMANDS].get(commandName);
      const rawArgv = this.rawArgv.slice();
      rawArgv.splice(rawArgv.indexOf(commandName), 1);

      const command = this.getSubCommandInstance(Command, rawArgv);
      // 获取egg-bin实例并执行其DISPATCH方法
      yield command[DISPATCH]();
      return;
    }

    const context = this.context;

    // print completion for bash
    if (context.argv.AUTO_COMPLETIONS) {
        ....
    } else {
      yield this.helper.callFn(this.run, [ context ], this);
    }
  }
```

直到所有this[DISPATCH]都执行完毕，才开始执行helper类继续执行每个command文件中的* run()函数，对于命令npm run dev而言，会处理this.helper.forkNode(this.serverBin, devArgs, options)，这部分主要运行的是egg-cluster中的startCluster,fork出agentWorker和appWorker。等到agentWorker和appWorker都初始化成功后，将会通知Master进程，然后Master进程再通知Agent和Worker应用启动成功。最后变回到了Koa中的入口文件的执行了。

本文章是在初次接触egg后的一些总结，如有错漏，欢迎指正。