---
title: Vue组件通信的六种方式
date: 2019-05-24 21:00:00
author: Echo
tags:
  - Vue
  - JavaScript
  - Compontent
---

在平时的开发过程中，父子 / 兄弟组件间的通信是肯定会遇到的啦，所以这里总结了 6 种 Vue 组件的通信方式：

##### 1. props / \$emit

##### 2. $emit / $on

##### 3. Vuex

##### 4. $attrs / $listeners

##### 5. $parent / $children 与 ref

##### 6. provide / inject

<!-- more -->

# 前言

<img src="/images/vue_compontent.png" width="400">
如上图所示，A/B，B/C，B/D组件是父子关系，C/D是兄弟关系。那如何根据不同的使用场景，选择不同的通信方式呢？所以前提就是我们要了解不同的通信方式的作用和区别。

# 一. props / \$emit

这个是我们平时用得比较多的方式之一，父组件 A 通过 props 参数向子组件 B 传递数据，B 组件通过$emit向A组件发送一个事件（携带参数数据），A组件中监听$emit 触发的事件得到 B 向 A 发送的数据。
我们来具体解释下它的实现步骤：

### 1：父组件向子组件传值

```js
// App.vue 父组件
<template>
    <a-compontent :data-a="dataA"></a-compontent>
</template>
<script>
import aCompontent from './components/A.vue';
export default {
    name: 'app',
    data () {
        return {
            dataA: 'dataA数据'
        }
    }
}
// aCompontent 子组件
<template>
    <p>{{dataA}}</p> // 在子组件中把父组件传递过来的值显示出来
</template>
<script>
export default {
    name: 'aCompontent',
    props: {
        dataA: {           //这个就是父组件中子标签自定义名字
            type: String,
            required: true  // 或者false
        }
    }
}
</script>
```

### 2：子组件向父组件传值（通过事件方式）

```js
// 子组件
<template>
    <p @click="sendDataToParent">点击向父组件传递数据</p>
</template>
<script>
export default {
    name: 'child',
    methods:{
        changeTitle() {
            this.$emit('sendDataToParent','这是子组件向父组件传递的数据'); // 自定义事件，会触发父组件的监听事件，并将数据以参数的形式传递
        }
    }
}

// 父组件
<template>
    <child @sendDataToParent="getChildData"></child>
</template>
<script>
import child from './components/child.vue';
export default {
    name: 'child',
    methods:{
        getChildData(data) {
            console.log(data); // 这里的得到了子组件的值
        }
    }
}
</script>
```

# 二. $emit / $on

这种方式是通过一个类似 App.vue 的实例作为一个模块的事件中心，用它来触发和监听事件，如果把它放在 App.vue 中，就可以很好的实现任何组件中的通信，但是这种方法在项目比较大的时候不太好维护。

举个 🌰：
假设现在有 4 个组件，Home.vue 和 A/B/C 组件，AB 这三个组件是兄弟组件，Home.vue 相当于父组件

```js
// 我们可以在router-view中监听change事件，也可以在mounted方法中监听
// home.vue
<template>
  <div>
    <child-a />
    <child-b />
    <child-c />
  </div>
</template>
```

```js
// A组件
<template>
  <p @click="dataA">将A组件的数据发送给C组件 - {{name}}</p>
</template>
data() {
  return {
    name: 'Echo'
  }
},
methods: {
  send() {
    this.$emit('data-a', this.name);
  }
}
```

```js
// B组件
<template>
  <p @click="dataB">将B组件的数据发送给C组件 - {{age}}</p>
</template>
data() {
  return {
    age: '18'
  }
},
methods: {
  send() {
    this.$emit('data-b', this.age);
  }
}
```

```js
// C组件
<template>
  <p>C组件得到的数据 {{name}} {{age}}</p>
</template>
data() {
  return {
    name: '',
    age: ''
  }
},
mounted() {
  // 在模板编译完成后执行
  this.$on('data-a',name => {
    this.name = name;
  })
  this.$on('data-b',age => {
    this.age = age;
  })
}
```

上面的 🌰 里我们可以知道，在 C 组件的 mounted 事件中监听了 A/B 的 \$emit 事件，并获取了它传递过来的参数（由于不确定事件什么时候触发，所以一般在 mounted / created 中监听）

# 三. Vuex

Vuex 是一个状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
Vuex 应用的核心是 store（仓库，一个容器），store 包含着你的应用中大部分的状态 (state)；

这个部分就不详细介绍了，官方文档很详细了 https://vuex.vuejs.org/zh/guide/state.html

# 四. $attrs / $listeners

<img src="/images/attrs.png" width="400">

如上图所示，这是一个多级组件的嵌套，那 A/C 组件如何进行通信？我们现在可以想到的有下面几种方案：

1. 使用 Vuex 来进行数据管理，但是使用的 vuex 的问题在于，如果项目比较小，组件间的共享状态比较少，那用 vuex 就好比杀鸡用牛刀。
2. 利用 B 组件做中转站，当 A 组件需要把信息传给 C 组件时，B 接受 A 组件的信息，然后用 props 传给 C 组件, 但是如果嵌套的组件过多，会导致代码繁琐，代码维护比较困难;如果 C 中状态的改变需要传递给 A, 还要使用事件系统一级级往上传递 。

在 Vue2.4 中，为了解决该需求，引入了$attrs 和$listeners ， 新增了 inheritAttrs 选项。(如下图所示)

<img src="/images/vue.png" width="600">
<img src="/images/inheritAttrs.png" width="580">

🌰：（\$attrs 的作用，某些情况下需要结合 inheritAttrs 一起使用）

有 4 个组件：App.vue / child1.vue / child2.vue / child3.vue，这 4 个组件分别的依次嵌套的关系。

```js
// App.vue
<template>
  <div id="app">
    <p>App.vue</p><hr>
    // 这里我们可以看到，app.vue向下一集的child1组件传递了5个参数，分别是name / age / job / sayHi / title
    <child1 :name="name" :age="age" :job="job" :say-Hi="say" title="App.vue的title"></child1>
  </div>
</template>
<script>
const child1 = () => import("./components/child1.vue");
export default {
  name: 'app',
  components: { child1 },
  data() {
    return {
      name: "Echo",
      age: "18",
      job: "FE",
      say: "this is Hi~"
    };
  }
};
</script>
```

```js
// child1.vue
<template>
  <div class="child1">
    <p>child1.vue</p>
    <p>name: {{ name }}</p>
    <p>childCom1的$attrs: {{ $attrs }}</p>
    <p>可以看到，$attrs这个对象集合中的值 = 所有传值过来的参数 - props中显示定义的参数</p>
    <hr>
    <child2 v-bind="$attrs"></child2>
  </div>
</template>
<script>
const child2 = () => import("./child2.vue");
export default {
  components: {
    child2
  },
  // 这个inheritAttrs默认值为true，不定义这个参数值就是true，可手动设置为false
  // inheritAttrs的意义在用，可以在从父组件获得参数的子组件根节点上，将所有的$attrs以dom属性的方式显示
  inheritAttrs: true, // 可以关闭自动挂载到组件根元素上的没有在props声明的属性
  props: {
    name: String // name作为props属性绑定
  },
  created() {
    // 这里的$attrs就是所有从父组件传递过来的所有参数 然后 除去props中显式定义的参数后剩下的所有参数！！！
    console.log(this.$attrs); //  输出{age: "18", job: "FE", say-Hi: "this is Hi~", title: "App.vue的title"}
  }
};
</script>

```

```js
// child2.vue
<template>
  <div class="child2">
    <p>child2.vue</p>
    <p>age: {{ age }}</p>
    <p>childCom2: {{ $attrs }}</p>
    <hr>
    <child3 v-bind="$attrs"></child3>
  </div>
</template>
<script>
const child3 = () => import("./child3.vue");
export default {
  components: {
    child3
  },
  // 将inheritAttrs设置为false之后，将关闭自动挂载到组件根元素上的没有在props声明的属性
  inheritAttrs: false,
  props: {
    age: String
  },
  created() {
    // 同理和上面一样，$attrs这个对象集合中的值 = 所有传值过来的参数 - props中显示定义的参数
    console.log(this.$attrs);
  }
};
</script>

```

```js
// child3.vue
<template>
  <div class="child3">
    <p>child3.vue</p>
    <p>job: {{job}}</p>
    <p>title: {{title}}</p>
    <p>childCom3: {{ $attrs }}</p>
  </div>
</template>
<script>
export default {
  inheritAttrs: true,
  props: {
    job: String,
    title: String
  }
};
</script>

```

来看下具体的显示效果：
<img src="/images/attrs_result.png" width="900">

而$listeners怎么用呢，官方文档说的是：包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用！
从字面意思来理解应该是在需要接受值的父组件增加一个监听事件？话不多说，上代码

还是 3 个依次嵌套的组件

```js
<template>
  <div class="child1">
    <child2 v-on:upRocket="reciveRocket"></child2>
  </div>
</template>
<script>
const child2 = () => import("./child2.vue");
export default {
  components: {
    child2
  },
  methods: {
    reciveRocket() {
      console.log("reciveRocket success");
    }
  }
};
</script>
```

```js
<template>
  <div class="child2">
    <child3 v-bind="$attrs" v-on="$listeners"></child3>
  </div>
</template>
<script>
const child3 = () => import("./child3.vue");
export default {
  components: {
    child3
  },
  created() {
    this.$emit('child2', 'child2-data');
  }
};
</script>
```

```js
// child3.vue
<template>
  <div class="child3">
    <p @click="startUpRocket">child3</p>
  </div>
</template>
<script>
export default {
  methods: {
    startUpRocket() {
      this.$emit("upRocket");
      console.log("startUpRocket");
    }
  }
};
</script>
```

这里的结果是，当我们点击 child3 组件的 child3 文字，触发 startUpRocket 事件，child1 组件就可以接收到，并触发 reciveRocket
打印结果如下：

```js
> reciveRocket success
> startUpRocket
```

# 五. $parent / $children 与 ref

- ref：如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例
- $parent / $children：访问父 / 子实例

这两种方式都是直接得到组件实例，使用后可以直接调用组件的方法或访问数据。

我们先来看个用 ref 来访问组件的 🌰：

```js
// child1子组件
export default {
  data() {
    return {
      title: 'Vue.js'
    };
  },
  methods: {
    sayHello() {
      console.log('child1!!');
    }
  }
};
```

```js
// 父组件
<template>
  <child1 ref="child1"></child1>
</template>
<script>
  export default {
    mounted () {
      const child1 = this.$refs.child1;
      console.log(child1.title);  // Vue.js
      child1.sayHello();  // 弹窗
    }
  }
</script>
```

# 六. provide/inject

provide/inject 是 Vue2.2.0 新增 API,这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。如果你熟悉 React，这与 React 的上下文特性很相似。

provide 和 inject 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中。

由于自己对这部分的内容理解不是很深刻，所以感兴趣的可以前往官方文档查看： https://cn.vuejs.org/v2/api/#provide-inject

# 总结

常见使用场景可以分为三类：

1. 父子通信：props / $emit；$parent / $children；$attrs/\$listeners；provide / inject API； ref
2. 兄弟通信：Vuex
3. 跨级通信：Vuex；$attrs/$listeners；provide / inject API
