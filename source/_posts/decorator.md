---
title: 装饰器模式
date: 2019-08-18 13:00:00
tags:
  - 设计模式
---

装饰器模式在设计模式属于结构型模式，就是原有的基础上增加一些功能。

装饰器（Decorator）是一种与类（class）相关的语法，用来注释或修改类和类方法。

<!-- more -->

## react 中的装饰器模式

之前没有刻意去学习设计模式相关的东西，其实常用的设计模式大家都有接触过而且会用，只是不知道它叫啥。就如现在说到的装饰器模式，初看感觉特别熟悉，react 的高阶组件和 react-redux 中的 connect 不就是装饰器模式吗。先来看下它在项目中是如何使用的。

```javascript
class Index extends React.Components {
  // ...
}

export default connect(state => ({
  visible: state.modal.login,
  loading: state.loading
}))(Index);

// 另外一种写法
@connect(state => ({
  visible: state.modal.login,
  loading: state.loading
}))
class Index extends React.Components {
  // ...
}
```

我们先来了解下装饰器模式，最后来看下 connect 是如何实现的。

## 简单的例子

装饰器模式在设计模式属于结构型模式，装饰器模式的特点就是：

- 为对象添加新的功能
- 不改变原有的结构和功能

```javascript
class Circle {
  draw() {
    console.log('画一个圆形');
  }
}

// 装饰器
class Decorator {
  constructor(circle) {
    this.circle = circle;
  }

  draw() {
    this.circle.draw();
    // 添加新功能
    this.setRedBorder();
  }

  setRedBorder() {
    console.log('设置红色边框');
  }
}

let circle = new Circle();
circle.draw();

let dec = new Decorator(circle);
dec.draw();
```

## ES7 中的装饰器

Decorator 提案经过了大幅修改，目前还没有定案，不过大家已经用了挺久了，先来看下用法。

### 装饰类

```javascript
// 装饰类
@testDec
class Demo {
  // ...
}

function testDec(target) {
  target.isDec = true;
}

alert(Demo.isDec);
```

```javascript
@decorator
class A {}

// 等同于
class A {}
A = decorator(A) || A;
```

testDec 就是一个装饰器。它修改了 Demo 这个类的行为，为它加上了静态属性 isDec。testDec 函数的参数target 是 Demo 类本身

### 传参数

```javascript
// mixin示例

function mixins(...func) {
  // 传参数需要返回一个函数
  return function (target) {
    Object.assign(target.prototype, ...list);
  }
}

const Foo = {
  foo() {
    alert('foo');
  }
}

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo(); // 'foo'
```

### 装饰方法

方法装饰器有固定的格式，第一个参数是类的原型对象，第二个参数是所要装饰的属性名，第三个参数是该属性的描述对象，Object.defineProperty 中会用到。即：

```javascript
{
  value: specifiedFunction,
  enumerable: false,
  configurable: true,
  writable: true
}
```

```javascript
function readonly(target, name, descriptor){
  descriptor.writable = false;
  return descriptor;
}

class Person {
  constructor() {
    this.first = 'hello';
    this.last = 'world';
  }

  @readonly
  name() {
    return `${this.first} ${this.last}`;
  }
}

let p = new Person();
p.name = function () {}; // 这里会报错，因为是只读的
```

### 装饰方法-添加log

是否遇到过这样的情景，比如使用 vue 框架的时候，使用了某个方法会在 console 打印一些日志，表示这个方法即将被弃用请使用新的方法替代。使用下面的装饰器就能很容易实现这个功能。

```javascript
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value;

  descriptor.value = function() {
    console.log('执行了加法');
    return oldValue.apply(this, arguments);
  };

  return descriptor;
}

const math = new Math();
math.add(2, 4);
```

## connect 是如何实现的

了解了装饰器是如何使用的，下面就可以来看一下 connect 是如何实现的了。

connect 是用来连接 Redux 和 React的，它包在我们的容器组件的外一层，它接收上面 Provider 提供的 store 里面的 state 和 dispatch，传给一个构造函数，返回一个对象，以属性形式传给我们的容器组件。connect 是一个高阶函数，首先传入mapStateToProps、mapDispatchToProps，然后返回一个生产 Component 的函数。

主要的逻辑如下：

```javascript
export default function connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {}) {
  return function wrapWithConnect(WrappedComponent) {
    class Connect extends Component {
      constructor(props, context) {
        // 从Component处获得store
        this.store = props.store || context.store;
        this.stateProps = computeStateProps(this.store, props);
        this.dispatchProps = computeDispatchProps(this.store, props);
        this.state = { storeState: null };
        // 对stateProps、dispatchProps、parentProps进行合并
        this.updateState();
      }

      shouldComponentUpdate(nextProps, nextState) {
        // 进行判断，当数据发生改变时，Component重新渲染
        if (propsChanged || mapStateProducedChange || dispatchPropsChanged) {
          this.updateState(nextProps);
          return true;
        }
      }

      componentDidMount() {
        // 改变Component的state
        this.store.subscribe(() => {
          this.setState({
            storeState: this.store.getState()
          });
        });
      }

      render() {
        // 生成包裹组件Connect
        return <WrappedComponent {...this.nextState} />;
      }
    }

    Connect.contextTypes = {
      store: storeShape
    };

    return Connect;
  };
}
```
