---
title: vuex 源码分析
date: 2019-12-16 20:56:44
tags: Vuex
---

## 前言
在平时开发中，经常会用`vuex`管理页面的数据，尤其是用户信息等一些全局的数据，觉得`vuex`使用很简单、方便，下面是官网给出的`vuex`的工作流程。

<!-- more -->

![][pic1]
其实还是很简单的，但是也有一些疑问：
- 为什么在`vue`的任何一个组件和页面都可以访问到`store`。
- `store`里面的数据是如何实现响应式的。
- 在我们使用`module`模式的时候，是如何实现模块嵌套的。
- 如何保证`store`里面的数据只能通过`commit`进行修改。

## 项目目录
```
.
├── LICENSE
├── README.md
├── build
├── dist
├── docs
├── docs-gitbook
├── examples
├── myTest
├── node_modules
├── package-lock.json
├── package.json
├── src
├── test
├── types
└── yarn.lock
```
主要的源码在`src`目录中，打开`src`目录
```
.
├── helpers.js
├── index.esm.js
├── index.js
├── mixin.js
├── module
│   ├── module-collection.js
│   └── module.js
├── plugins
│   ├── devtool.js
│   └── logger.js
├── store.js
└── util.js
```
## 找到入口文件
打开`package.json`，看一下打包的命令:
```
"dev:dist": "rollup -wm -c build/rollup.dev.config.js",
```
其实是运行了`build/rollup.dev.config.js`文件，打开`build/rollup.dev.config.js`文件，看到:
```
const { input, output } = require('./configs').commonjs

module.exports = Object.assign({}, input, { output })
```
`input` 其实是在`configs`中，打开`configs`文件
```
input: resolve('src/index.js'),
```
发现入口文件是`src/index.js`，打开`src/index.js`
```
import { Store, install } from './store'
import { mapState, mapMutations, mapGetters, mapActions, createNamespacedHelpers } from './helpers'

export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}
```
## 怎么个流程？
我们使用的时候，一般是以下这样的:
```
import Vuex from 'vuex';

vue.use(Vuex);

const store = new Vuex.Store();
```
让我们看一下`vue.use(Vuex)`发生了啥？
```
import { Store, install } from './store'
import { mapState, mapMutations, mapGetters, mapActions, createNamespacedHelpers } from './helpers'

export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}
```
`vue.use`很明显是`vue`的一个全局方法，我们打开`vue`的`github`地址，在`vue/src/core/global-api/use.js`下找到`vue.use`的源码
```
/* @flow */

import { toArray } from '../util/index'

export function initUse (Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this)
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
}
```
简单分析以下：
- `vue.use`需要传入一个参数，可以是一个`function`或者是一个`object`。
- 声明一个变量`installedPlugins`来存储安装的插件，默认为空数组。
- 判断传入的插件是否已经存在了，如果已存在，则直接返回。
- 获取传入的多余的参数，如果`vue.use(plugin, 1, 2, 3)`，则args获取到的是`[1,2,3]`。
- 把this添加到参数数组的首部。
- 如果插件是一个对象，并且这个对象有`install`方法，则调用`install`方法，并传入参数，如果插件是一个函数，则直接调用，并传入参数。
- 把插件`push`到`installedPlugins`中。
- 返回。

## 为什么在vue的任何一个组件和页面都可以访问到store
可以看到，其实vue.use调用的vuex的install方法
```
import { Store, install } from './store'
```
install 方法在store.js中，打开store.js，可以看到install源码如下：
```
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```
源码看起来很简单，就是判断`vuex`是否已经被`use`了，如果已经`use`了，在生产环境给出提示，然后调用了`appMixin`方法，打开`appMixin`源码
```
export default function (Vue) {
  const version = Number(Vue.version.split('.')[0])

  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit
      _init.call(this, options)
    }
  }

  /**
   * Vuex init hook, injected into each instances init hooks list.
   */

  function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
```
从上面可以看到，首先判断`vue`的版本，如果是`vue`的版本大于等于`2`，则在生命周期`beforeCreate`中初始化`vuex`，如果为`vue1.0`的版本，则重写`_init`方法进行注入。
从源码中可以看出:
```
// store injection
 ...
  } else if (options.parent && options.parent.$store) {
    this.$store = options.parent.$store
  }
```
通过实现把`store`注入到每一个`vue`组件和页面中。

## store里面的数据是如何实现响应式的
在`store.js` 文件中
```
function resetStoreVM (store, state, hot) {
  const oldVm = store._vm

  // bind store public getters
  store.getters = {}
  // reset local getters cache
  store._makeLocalGettersCache = Object.create(null)
  const wrappedGetters = store._wrappedGetters
  const computed = {}
  forEachValue(wrappedGetters, (fn, key) => {
    // use computed to leverage its lazy-caching mechanism
    // direct inline function use will lead to closure preserving oldVm.
    // using partial to return function with only arguments preserved in closure environment.
    computed[key] = partial(fn, store)
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key],
      enumerable: true // for local getters
    })
  })

  // use a Vue instance to store the state tree
  // suppress warnings just in case the user has added
  // some funky global mixins
  const silent = Vue.config.silent
  Vue.config.silent = true
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
  })
  Vue.config.silent = silent

  // enable strict mode for new vm
  if (store.strict) {
    enableStrictMode(store)
  }

  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      store._withCommit(() => {
        oldVm._data.$$state = null
      })
    }
    Vue.nextTick(() => oldVm.$destroy())
  }
}
```
上面代码主要做了：
- 存储当前实例。
- 在`store`上增加一个空的`getters`对象。
- 在`store`上增加`_makeLocalGettersCache`缓存`store`数据。
- 把当前实例的`store`中的代理到`store`的`getters`中，并设置为可枚举的方式。
- 实例化一个`vue`实例，把`state`声明为该实例`data`中的一个变量（重点）。
- 销毁`oldVm`。

从上面的代码可以看出，`vuex`实现响应式的原理是借用了`vue`的响应式原理，以下是`vue`中实现响应式的代码：
```
 Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) return
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
```
从上面代码可以看出，其实`vue`实现响应式的原理就是使用了`Object.defineProperty`方法，这个方法没办法使用`ployfill`去实现，这也是为什么`vue`不支持`ie8`的主要原因


## 在我们使用module模式的时候，是如何实现模块嵌套的
模块嵌套的实现，主要是以下源码
```
function installModule (store, rootState, path, module, hot) {
  const isRoot = !path.length
  const namespace = store._modules.getNamespace(path)
  // register in namespace map
  if (module.namespaced) {
    if (store._modulesNamespaceMap[namespace] && process.env.NODE_ENV !== 'production') {
      console.error(`[vuex] duplicate namespace ${namespace} for the namespaced module ${path.join('/')}`)
    }
    store._modulesNamespaceMap[namespace] = module
  }

  // set state
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      if (process.env.NODE_ENV !== 'production') {
        if (moduleName in parentState) {
          console.warn(
            `[vuex] state field "${moduleName}" was overridden by a module with the same name at "${path.join('.')}"`
          )
        }
      }
      Vue.set(parentState, moduleName, module.state)
    })
  }

  const local = module.context = makeLocalContext(store, namespace, path)

  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })

  module.forEachAction((action, key) => {
    const type = action.root ? key : namespace + key
    const handler = action.handler || action
    registerAction(store, type, handler, local)
  })

  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })

  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
}
```
从源码中可以很清晰的看出，即使我们写的时候，是嵌套的，实际上，`vuex`帮我们拉平了，放在`_modulesNamespaceMap`中，拉平的实现，主要是
```
...
module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
})
...
```
通过递归调用的方式实现。

## 如何保证store里面的数据只能通过commit进行修改
主要通过对`state`执行`$watch`来禁止从`mutation`外部修改`state`
```
function enableStrictMode (store) {
  store._vm.$watch(function () { return this._data.$$state }, () => {
    if (process.env.NODE_ENV !== 'production') {
      assert(store._committing, `do not mutate vuex store state outside mutation handlers.`)
    }
  }, { deep: true, sync: true })
}
```
所有的修改，在`vuex`的内部会调用一个`_withCommit`私有方法，通过外部直接修改，没办法调用这个私有方法，在开发环境会抛出错误
```
_withCommit (fn) {
    const committing = this._committing
    this._committing = true
    fn()
    this._committing = committing
  }
```

## 总结
总的来说，`vuex`的源码结构还是比较清晰，容易阅读的，特别适合我小白，比如我😊，欢迎探讨。


[pic1]:data:image/png;base64,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FCERIECAAAECBAjkFZD05vVUGgECBAgQIECAQIECkt4CG0VIBAgQIECAAAECeQUkvXk9lUaAAAECBAgQIFCggKS3wEYREgECBAgQIECAQF4BSW9eT6URIECAAAECBAgUKCDpLbBRhESAAAECBAgQIJBXQNKb11NpBAgQIECAAAECBQpIegtsFCERIECAAAECBAjkFZD05vVUGgECBAgQIECAQIECkt4CG0VIBAgQIECAAAECeQUkvXk9lUaAAAECBAgQIFCggKS3wEYREgECBAgQIECAQF4BSW9eT6URIECAAAECBAgUKCDpLbBRhESAAAECBAgQIJBXQNKb11NpBAgQIECAAAECBQpIegtsFCERIECAAAECBAjkFZD05vVUGgECBAgQIECAQIECkt4CG0VIBAgQIECAAAECeQUkvXk9lUaAAAECBAgQIFCggKS3wEYREgECBAgQIECAQF4BSW9eT6URIECAAAECBAgUKCDpLbBRhESAAAECBAgQIJBXQNKb11NpBAgQIECAAAECBQpIegtsFCERIECAAAECBAjkFZD05vVUGgECBAgQIECAQIECkt4CG0VIBAgQIECAAAECeQUkvXk9lUaAAAECBAgQIFCggKS3wEYREgECBAgQIECAQF4BSW9eT6URIECAAAECBAgUKCDpLbBRhESAAAECBAgQIJBXQNKb11NpBAgQIECAAAECBQpIegtsFCERIECAAAECBAjkFZD05vVUGgECBAgQIECAQIECkt4CG0VIBAgQIECAAAECeQUkvXk9lUaAAAECBAgQIFCggKS3wEYREgECBAgQIECAQF4BSW9eT6URIECAAAECBAgUKCDpLbBRhESAAAECBAgQIJBXQNKb11NpBAgQIECAAAECBQpIegtsFCERIECAAAECBAjkFZD05vVUGgECBAgQIECAQIECkt4CG0VIBAgQIECAAAECeQUkvXk9lUaAAAECBAgQIFCggKS3wEYREgECBAgQIECAQF4BSW9eT6URIECAAAECBAgUKCDpLbBRhESAAAECBAgQIJBXQNKb11NpBAgQIECAAAECBQpIegtsFCERIECAAAECBAjkFZD05vVUGgECBAgQIECAQIECkt4CG0VIBAgQIECAAAECeQUkvXk9lUaAAAECBAgQIFCggKS3wEYREgECBAgQIECAQF4BSW9eT6URIECAAAECBAgUKCDpLbBRhESAAAECBAgQIJBXQNKb11NpBAgQIECAAAECBQpIegtsFCERIECAAAECBAjkFZD05vVUGgECBAgQIECAQIECkt4CG0VIBAgQIECAAAECeQUkvXk9lUaAAAECBAgQIFCggKS3wEYREgECBAgQIECAQF4BSW9eT6URIECAAAECBAgUKCDpLbBRhESAAAECBAgQIJBX4LyJiYmvpJSuylus0ggQIECAAAECBAgUI7D1/wEWU54M3ToqTQAAAABJRU5ErkJggg==

