---
title: table 组件了解一下？
date: 2019-11-12 18:24:51
tags:
  - element-ui
  - table
  - 组件
  - 源码
---

## 前言

国庆去了一趟西藏，那边的风景很赞 👍，但是天天都会头疼 😣，不禁感慨，还是写文章好啊 ✍，So，今天要和大家分享的是 table 组件的实现，是从 0 到 1 的实现哦，这个组件对于我们来说应该是挺复杂的一个了，看过那么多个初级组件，是时候装个叉了 😏。

<!-- more -->

## 知识回顾

表格这东西我们肯定都接触过，尤其是在开发后台管理系统的时候，不过大部分都是直接用 UI 框架写的，久了都不知道原来是怎么写的了，所以在此我们先回顾一下 🤔 最原始的表格的基础写法：

```html
<table border="1">
  <colgroup>
    <!-- 这里可以针对每列做一些属性设置，例如设置宽度，这个我还真不知道，也许曾经看过但忘了 -->
    <col width="200" />
    <col width="150" />
    <col width="100" />
  </colgroup>
  <thead>
    <tr>
      <th>姓名</th>
      <th>职位</th>
      <th>等级</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>尤水就下</td>
      <td>前端</td>
      <td>小菜</td>
    </tr>
  </tbody>
</table>
```

上面的表格渲染出来大概是下面这个样子：
![](https://user-gold-cdn.xitu.io/2019/10/31/16e1fc5458a19564?w=946&h=218&f=png&s=32645)
其中要注意的就是 `<col>` 了，因为这个东西对我来说是挺陌生的（其实我不知道
😂），然后就搜了一下，原来它是方便我们给每个列设置一些共同属性的，比如宽度。因为后续我们会用到它，所以请务必记住 👀。

## 目标

首先简要说下我们本篇文章要实现的东西：基础展示 + 全选 + 排序 + 展开 + 自定义内容 + 固定表头 +（多级表头 + 固定列）。然后话不多说，撸起袖子就是干 💪。

## 基础展示

作为一个表格，展示数据是必备的功能了，当然它很容易实现，但更为重要的是 api 的设计，一个好的 api 能够让你事半功倍，所以在借鉴了各大框架的 api 之后，我们希望开发是这样使用我们组件的 👇：

```html
<template>
  <div id="app">
    <xr-table :columns="columns" :data="data"></xr-table>
  </div>
</template>
<script>
  import XrTable from "./components/xr-table";
  export default {
    components: {
      XrTable
    },
    data() {
      return {
        columns: [
          {
            title: "姓名",
            key: "name"
          },
          {
            title: "年龄",
            key: "age"
          },
          {
            title: "职位",
            key: "job"
          }
        ],
        data: [
          {
            id: 1,
            name: "Jasmine",
            age: 18,
            job: "产品",
            desc: "这是展开的描述啊1"
          },
          {
            id: 2,
            name: "Mango",
            age: 18,
            job: "设计",
            desc: "这是展开的描述啊2"
          },
          {
            id: 3,
            name: "Aking",
            age: 24,
            job: "前端",
            desc: "这是展开的描述啊3"
          },
          {
            id: 4,
            name: "Dick",
            age: 30,
            job: "后端",
            desc: "这是展开的描述啊4"
          },
          {
            id: 5,
            name: "Lucy",
            age: 18,
            job: "测试",
            desc: "这是展开的描述啊5"
          }
        ]
      };
    }
  };
</script>
```

也就是你给负责给数据（`columns` 要求有要有 `key`、`data` 要求要有 `id`），我来渲染，这部分其实没有难度，加上点样式表格就挺漂亮了，这里就直接上代码了：

```html
<template>
  <div class="xr-table">
    <table>
      <thead>
        <tr>
          <!-- 表头循环 -->
          <th v-for="col in columns" :key="col.key">{{col.title}}</th>
        </tr>
      </thead>
      <tbody>
        <!-- 表体循环 -->
        <tr v-for="row in data" :key="row.id">
          <td v-for="col in columns" :key="col.key">{{row[col.key]}}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<style lang="scss">
  .xr-table {
    table {
      width: 100%;
      border-collapse: collapse;
      border-spacing: 0;
      empty-cells: show;
      border: 1px solid #e9e9e9;
    }
    table th {
      background: #f7f7f7;
      color: #5c6b77;
      font-weight: 600;
      white-space: nowrap;
    }
    table td,
    table th {
      padding: 8px 16px;
      border: 1px solid #e9e9e9;
      text-align: left;
    }
  }
</style>
```

其实就是把 `th` 和 `td` 循环一遍，应该不用过多解释 😁。然后运行代码，看下效果：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2af503aaa59e3?w=1026&h=504&f=png&s=45589)
这样，一个简简单单的表格就有了，当然了，我们的目标不止于此。

## 全选

接下来我们需要给表格添加全选功能，首先还是看 api 咋弄，经过参考之后，我们可以在传进来的 `columns` 里面做文章，并在勾选的时候触发一个 `on-selection-change` 事件，就像下面这样 👇：

```html
<template>
  <xr-table
    :columns="columns"
    :data="data"
    @on-selection-change="onSelectionChange"
  ></xr-table>
</template>
<script>
  export default {
      data() {
          return {
              columns: [
                  {
                      type: 'selection' // 这个地方可以不用写 key，type 就相当于 key
                  }
                  ...
              ]
          }
      }
  }
</script>
```

说实话我觉得这个 api 设计是挺巧妙的，不需要我们像 Element 那样写：

```html
<el-table>
  <el-table-column type="selection" width="55"> </el-table-column>
</el-table>
```

然后组件里面该怎么改呢？也很简单，在循环 `th` 和 `td` 的时候先判断是否有 `type`，如果有 `type` 并且值为 `selection` 就渲染成复选框，就像下面这样 👇：

```html
<template>
  <div class="xr-table">
    <table>
      <thead>
        <tr>
          <th v-for="col in columns" :key="col.key">
            <div>
              <!-- 在这里先判断 type -->
              <template v-if="col.type === 'selection'">
                <input
                  ref="allCheckbox"
                  type="checkbox"
                  :checked="isSelectAll"
                  @change="selectAll"
                />
              </template>
              <template v-else>{{col.title}}</template>
            </div>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in data" :key="row.id">
          <td v-for="col in columns" :key="col.key">
            <div>
              <!-- 在这里先判断 type -->
              <template v-if="col.type === 'selection'">
                <input
                  type="checkbox"
                  :checked="formateStatus(row)"
                  @change="toggleSelect($event, row)"
                />
              </template>
              <template v-else>{{row[col.key]}}</template>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
```

当然了，上面的代码只是把复选框渲染出来而已，现在我们还需要为其加上点击的逻辑。这里我们在组件里面维护了一个 `selectedRows` 字段，我们把当前的已选行都放入到这个数组中来，每次点击复选框都会修改 `selectedRows` 的值。要注意的是表头复选框的值 `isSelectAll` 是根据所有数据 `data` 和已选行 `selectedRows` 进行比较得到的，所以 `isSelectAll` 应该写在计算属性里面，就像下面这样 👇：

```html
<script>
  export default {
    ...
    data() {
      return {
        selectedRows: [] // 当前已选中的行
      };
    },
    computed: {
      isSelectAll() { // 表头全选的勾选状态应该根据当前已选的来计算，最好不要直接比较数组长度是否相等，而是应该在比较长度的基础上比较每一项的 id 是否一样，虽然目前看起来这个步骤很多余
        let all = this.data.map(item => item.id).sort();
        let selected = this.selectedRows.map(item => item.id).sort();
        let isSelectAll = true;
        if (all.length === selected.length) {
          for (let i = 0, len = all.length; i < len; i++) {
            if (all[i] !== selected[i]) {
              isSelectAll = false;
              break;
            }
          }
        } else {
          isSelectAll = false;
        }
        this.$nextTick(() => { // 这个是选了部分之后把表头的复选框改变成中间状态（就是横杠的状态）
          this.$refs['allCheckbox'][0].indeterminate =
            selected.length && !isSelectAll;
        });
        return isSelectAll;
      }
    },
    methods: {
      selectAll(e) { // 单击表头的多选框并向外触发事件
        let checked = e.target.checked;
        this.selectedRows = checked ? JSON.parse(JSON.stringify(this.data)) : [];
        this.$emit('on-selection-change', this.selectedRows);
      },
      toggleSelect(e, row) { // 单击表体的多选框并向外触发事件
        let checked = e.target.checked;
        if (checked) {
          this.selectedRows.push(row);
        } else {
          let idx = this.selectedRows.findIndex(item => item.id === row.id);
          this.selectedRows.splice(idx, 1);
        }
        this.$emit(
          'on-selection-change',
          JSON.parse(JSON.stringify(this.selectedRows))
        );
      },
      formateStatus(row) { // 表体的每个多选框是否被勾选
        return this.selectedRows.findIndex(item => item.id === row.id) >= 0;
      }
    }
  };
</script>
```

上面代码里面的注释应该解释的还算清楚，下面看下实现的效果：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2afa27de967ee?w=600&h=236&f=gif&s=3383286)
应该还行吧！

## 排序

现在我们需要给表格加上排序功能，还是照旧先看一下 api 的设计，其实它和全选功能异曲同工，我们还是在 `columns` 里面做文章，然后支持向外触发`on-sort`事件即可，就像下面这样 👇：

```html
<template>
  <div id="app">
    <xr-table
      :columns="columns"
      :data="data"
      @on-sort="onSort"
    ></xr-table>
  </div>
</template>
<script>
export default {
  ...
  data() {
    return {
      columns: [
        ...
        {
          title: '年龄',
          key: 'age',
          sortable: true
        }
        ...
      ]
    }
  }
}
```

不过这里我们并没有真正的进行排序操作 😬，因为排序更应该是偏向让后端做，前端负责触发事件、调用接口、得到数据、刷新表格即可，毕竟单纯的前端排序没有什么太大意义。那什么时候可能需要前端排序呢，就是数据量不大，前端一次拿到所有数据的情况下，这样可以省去调用接口的时间，不过能推给后端就推吧 😏，下面我们来看下代码实现：

```html
...
<template v-if="col.type === 'selection'">
  <input
    ref="allCheckbox"
    type="checkbox"
    :checked="isSelectAll"
    @change="selectAll"
  />
</template>
<template v-else>
  <!-- 改动在这里 -->
  <span>{{col.title}}</span>
  <span v-if="col.sortable">
    <i @click="handleSort(col.key, 'asc')">↑</i>
    <i @click="handleSort(col.key, 'desc')">↓</i>
  </span>
</template>
...
<script>
  export default {
      ...
      methods: {
          handleSort(key, sortType) {
            this.$emit('on-sort', { key, sortType });
          }
      }
  }
</script>
```

虽然很简单，但是这里写排序的主要目的是，让我们的表格支持在表头增加一些图标和自定义事件，其运行结果如下：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2afa67a28484a?w=599&h=216&f=gif&s=2947993)
也还 ok 吧！

## 展开

展开其实和上面的两个功能一样，api 需要在 `columns` 里面做文章：

```html
<script>
export default {
  ...
  data() {
    return {
      columns: [
        {
          type: 'expand'
        }
        ...
      ]
    }
  }
}
```

这里我们打算维护一个 `expandIds`，保存所有展开行的信息，和全选的 `selectedRows` 一毛一样。但其实也可以有另一种做法：就是我们预先把传进来的数据处理一下，在 `data` 里面的每一行加上 `isExpand` 和 `isSelect` 字段，然后操作的时候进行相应的状态修改即可，就不需要再额外声明数组了。但是怎么实现我不管，做出来才是硬道理，所以请看下面的代码吧 👇：

```html
...
<template v-if="col.type === 'selection'">
  <input
    ref="allCheckbox"
    type="checkbox"
    :checked="isSelectAll"
    @change="selectAll"
  />
</template>
<!-- 改动在这里 -->
<template v-else-if="col.type === 'expand'"></template>
...
<tbody>
  <template v-for="row in data">
    <tr :key="row.id">
      <td v-for="col in columns" :key="col.key">
        ...
      </td>
    </tr>
    <!-- 这里多加了一个是否展开的判断 -->
    <tr :key="`expand-${row.id}`" v-if="checkIsExpand(row.id)">
      <!-- 横跨所有列 -->
      <td :colspan="columns.length">{{row.desc}}</td>
    </tr>
  </template>
</tbody>
...
<script>
  export default {
      ...
      data() {
          return {
            expandIds: []
          };
        },
      methods: {
          ...
          toggleExpand(id) {
            let idx = this.expandIds.indexOf(id);
            if (idx >= 0) {
              this.expandIds.splice(idx, 1);
            } else {
              this.expandIds.push(id);
            }
          },
          checkIsExpand(id) {
            return this.expandIds.indexOf(id) >= 0;
          }
      }
  }
</script>
```

要注意的是展开的内容是需要横跨所有行的，所以我们得把 `colspan` 的值设置成 `columns.length`，也就是横跨所有列的意思。当然目前我们是写死了展开的字段内容 `desc`，一会我们会支持自定义。这里我们也不注重样式，毕竟不是重点，然后运行一下，看下效果 👇：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2b33a792b069c?w=600&h=481&f=gif&s=110790)
好像有点意思！

## 自定义内容

说讲就讲 👏，写到这里其实我们已经支持最基础的表格需要，但问题是现在的表格也太基础了吧，要是我想自定义内容咋办（挠头三连 😧），好歹支持一下吧。所以接下来我们需要 do it！
首先想都不用想，这个东西肯定是要用插槽的，但我们还是要从 api 的设计入手，现在假设我们要在年龄这一列的后面加上一个“岁”字，并且在最后一列增加编辑和删除两个按钮，我们期待的应该是这样的用法 👇：

```html
<template>
    <xr-table
      :columns="columns"
      :data="data">
      <!-- 其中 age 是对应的插槽名，{row, col, index} 对应的是行、列、索引这三个参数 -->
      <template v-slot:age="{row, col, index}">{{ row.age + '岁'}}</template>
      <template v-slot:action="{row, col, index}">
        <button>编辑{{index}}</button>
        <button>删除</button>
      </template>
    </xr-table>
</template>
<script>
export default {
  ...
  data() {
    return {
      columns: [
        ...
        {
          title: '年龄',
          slot: 'age', // 写了 slot 也可以不用写 key，因为它相当于 key
          sortable: true
        },
        ...
        {
          title: '操作',
          slot: 'action'
        }
      ]
    }
    ...
}
```

我们在 `columns` 里面加了个 `slot` 字段，它用来表明表格的某一列是否需要自定义内容，然后在 `<xr-table></xr-table>` 里面写了个形如 `<template v-slot:age="{row, col, index}">{{ index }}</template>` 这样的一个东西，其中 `age` 是对应的插槽名，`{row, col, index}` 对应的是行、列、索引这三个参数，不知道大家对这个东西熟不熟悉，它其实就是 `slot-scope` 的新写法，让我们来看下官网的说明（不了解的同学可以去看下使用方法，并不难）：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2b1a2ab09f83c?w=1360&h=188&f=png&s=78930)
`v-slot` 插槽真是个好用的东西，本来写自定义内容还是挺繁琐的，`v-slot` 让实现变得简单了，此外它还有个简写方式，可以用 `#` 代替 `v-slot`，就像 `@` 代替 `v-on` 一样，也就是 `<template #age="slotProps">{{ index }}</template>`。
好了，现在我们来看下组件里的代码具体怎么写，其实只需要对 `tbody` 里面进行更改即可，没有想象中那么难，因为改动并不大，所以这里直接上代码：

```html
<tbody>
  <template v-for="(row, index) in data">
    <tr :key="row.id">
      <td v-for="col in columns" :key="col.key">
        <div>
          <!-- 改动在这里，我们我先判断列是否有 slot 字段 -->
          <template v-if="col.slot">
            <!-- row，col，index 是我们需要主动传出去的参数，这样在外面的 slotProps 才能拥有这几个参数，当然我们还可以传其他参数，他们最终都会被放在 slotProps 这一个对象里面 -->
            <slot :name="col.slot" :row="row" :col="col" :index="index"></slot>
          </template>
          <template v-else-if="col.type === 'selection'">
            ...
          </template>
          ...
        </div>
      </td>
    </tr>
    ...
  </template>
</tbody>
```

最后渲染出来就是我们要的模样了，如下图所示 👇：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2b9a8426e2e2e?w=1050&h=508&f=png&s=84139)
同样的道理，我们也可以稍稍修改下展开行使之能够支持自定义，大家可以尝试一下，这里就不做展示罗 😁。

## 固定表头

所谓固定表头就是表头不动，但表体可以滚动，这个东西实现起来就有点小麻烦了 🤔，为什么呢？因为本来我们的表格刚好是由 `thead` 和 `tbody` 两部分组成，常规的思路就是把 `tbody` 给个高度，然后让它溢出滚动即可，但是事情没那么简单，`height` 和 `overflow` 这两个样式写在 `tbody` 或者 `table` 上都是木有效果滴，So，我们就得去看看人家是咋做的啦（以 Element 为例）：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2bdf7c0989329?w=1674&h=570&f=png&s=196479)
通过上面这张图我们能够清晰的看到 `thead` 和 `tbody` 分别被放在了两个不同的 `div` 的 `table` 里面，然后设置 `tbody` 的外层 `div` 可以滚动就行了。嗯，真实直白的想法 😯。其实其它几大 UI 框架也是一样的（就是分开写）。那么既然人家已经实践过了，说明该方案是可行的，至少兼容性是杠杆的，于是我们在集百家之长之后 😬，就可以开始动手写属于自己的代码了。当然了，第一步还是 api 的设计，这回我们只需要在标签里传入 `height` 参数即可：

```html
<xr-table :columns="columns" :data="data" height="150"></xr-table>
```

接下来我们需要改变一下组件的内部结构，把它转换成两段式的写法。事实上表格的 `thead` 和 `tbody` 本来就是分开的，所以拆出来并不难，就像下面这样 👇：

```html
<template>
  <!-- 这里只是单纯改了结构，里面的 tr 并没有改变 -->
  <div class="xr-table">
    <div class="xr-table__header">
      <table>
        <thead>
          ...
        </thead>
      </table>
    </div>
    <div class="xr-table__body">
      <table>
        <tbody>
          ..
        </tbody>
      </table>
    </div>
  </div>
</template>
```

保存一下代码，目前效果图如下：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2bfae4147e6be?w=1104&h=512&f=png&s=84878)
嗯，和原来的并没有什么差别，但是第一个问题来了，`thead` 的宽度和 `tbody` 的宽度对不齐了，这可咋整啊。还能咋整啊，就定宽呗 😳，通过 `columns` 传入每列的 `width` 值就好了（当然可以有一列是可以不用给宽度的，这样可以保持弹性），然后把 `thead` 和 `tbody` 的每列设置成一样宽就行了，你可能会觉得麻烦，但事实上这能很好的避免其他一些不必要的问题，这里我给大家截了 Ant Design 和 Element 两个官网上的图：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2c04d55fef882?w=1732&h=294&f=png&s=78939)
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2c03f3b42815f?w=414&h=964&f=png&s=95151)
你可以清楚的看到他们也是需要 `width` 的，所以这里我们需要给 `columns` 多加上一个字段 `width`，形如这样 `{title: '姓名', key: 'name', width: 100}`，这里的单位默认是 `px`。然后怎么设置宽度呢？哈哈 😁，这里就要用到我们在开篇提到的 `<colgroup>` 里面的 `<col>` 啦，这个东西用来设置宽度实在是恰好不过了。组件里要改的地方也不多，把宽度加上就行了，就像下面这样 👇：

```html
<div class="xr-table__header">
  <table>
    <colgroup>
      <col v-for="col in columns" :width="col.width || ''" />
    </colgroup>
    ...
  </table>
</div>
<div class="xr-table__body">
  <table>
    <colgroup>
      <col v-for="col in columns" :width="col.width || ''" />
    </colgroup>
    ...
  </table>
</div>
```

这里我们把倒数第二列的宽度留空，然后看一下效果：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2c1b9e0391f83?w=599&h=233&f=gif&s=3976033)
恩，好像挺好 👏，那就继续吧。接下来要做的就是让表体定高并滚动了。首先我们传进来的 `height` 应该是整个表格的高度，所以要在最外层的 `xr-table` 要加上 `height: 150px; overflow: hidden` 的样式，然后需要计算并设置 `xr-table__body` 的高度（总高 - 表头）以及 `overflow` 的值，这样表体就能滚动了。具体看下面的代码，其实改动也不多：

```html
<template>
  <!-- 加了个 tableStyle -->
  <div class="xr-table" :style="tableStyle">
    <!-- 加了个 ref -->
    <div class="xr-table__header" ref="tableHeader">...</div>
    <div class="xr-table__body" ref="tableBody">...</div>
  </div>
</template>
<script>
  export default {
      ...
      mounted() {
          let { tableHeader, tableBody } = this.$refs;
          let headerH = parseInt(window.getComputedStyle(tableHeader).height);
          let bodyH = this.height - headerH;
          tableBody.style.height = `${bodyH}px`;
      },
      computed: {
          tableStyle() {
            return this.height ? `height: ${this.height}px` : '';
          }
      }
      ...
  }
</script>
<style lang="scss">
  .xr-table {
    overflow: hidden;
    &__body {
      overflow: auto;
    }
  }
</style>
```

保存运行一下，效果如下：
![](https://user-gold-cdn.xitu.io/2019/11/2/16e2c34660b7703d?w=599&h=146&f=gif&s=2798661)
嗯，也还不错，虽然样式有点瑕疵，但这不重要，毕竟功能已经实现了 ✌。需要看源代码的可以狠狠点击这里：[table 组件](https://github.com/lgq627628/xr-wheel/blob/master/src/table/table2.vue)。
**six six six，大赞无疆** 👍👍👍

## 结语

写到这里，可能内容有点多了，所以我们把多级表头和固定列留到下一篇讲，希望这个月下旬能写好吧 😂。虽然写的东西比较简单，不过讲的是从 0 开始的过程，希望能对大家有所帮助，回见 👋。
