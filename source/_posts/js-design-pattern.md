---
title: 浅谈JS设计模式
date: 2019-04-27 14:00:00
tags: 设计模式
---

最近复习了下JS的设计模式和其思想，所以今天对其做一些简单的总计
<!-- more -->

## unix/linux设计哲学

在复习设计模式之前，先给大家讲讲《unix/linux设计哲学》这本书里的一些理论，我想，通过这本书里的一些介绍，对我们去理解和明白设计模式有着更好的帮助

#### 首先总结书中以下的一些结论：

> * 小即是美
> * 让每个程序只做好一件事
> * 快速建立原型
> * 舍弃高效率而取可移植性
> * 采用纯文本来存储数据
> * 充分利用软件的杠杆效应（软件复用）
> * 使用shell脚本来提高杠杆效应和可移植性
> * 避免强制性的用户界面
> * 让每个程序都成为过滤器


###  其次是几个小准则
> * 小准则：允许用户定制环境
> * 小准则：尽量使操作系统内核小而轻量化
> * 小准则：使用小写字母并尽量简写
> * 小准则：沉默是金
> * 小准则：各部分之和大于整体
> * 小准则：寻求90%的解决方案

### 最后看一下SOLID五大设计原则

> * S - 单一职责原则
> * O - 开放封闭原则 
对扩展开放，对修改封闭
增加需求时，扩展新代码，而非修改已有代码
> * L - 李氏置换原则
子类能覆盖父类
父类能出现的地方子类就能出现
> * I - 接口独立原则
保持接口的单一独立，避免出现‘胖接口’
> * D - 依赖导致原则
面向接口编程，依赖于抽象而不依赖于具体
使用方只关注接口而不关注具体类的实现

## 进入正式论题

### 工厂模式

定义一个用于创建对象的接口，这个接口由子类决定实例化哪一个类。该模式使一个类的实例化延迟到了子类。而子类可以重写接口方法以便创建的时候指定自己的对象类型

<details>
    <summary>简单实现代码</summary>

    ```js
    class Product {
      constructor (name) {
        this.name = name;
      }
      init () {
        alert('init');
      }
      func1 () {
        alert('func1');
      }
    }

    class Creator {
      create (name) {
        return new Product(name);
      }
    }

    // test
    let creator = new Creator();
    let p = creator.create('p1');
    p.init();
    p.func1();

    ```
</details>

### 实际的应用场景
 
> * jQuery - $('div')
> * React.createElement
> * vue 异步组件

### 总结
主要好处在于消除对象间的耦合，防止代码的重用，有助于创建模块化的代码


### 单例模式

保证一个类仅有一个实例，并提供一个访问它的全局访问点。

<details>
    <summary>简单实现代码</summary>

    ```js
    class SingleObject {
      login () {
        console.log('login ...')
      }
    }

    SingleObject.getInstance = (function () {
      let instance;
      return function () {
        if (!instance) {
          instance = new SingleObject();
        }
        return instance;
      }
    })();

    // test
    let obj1 = SingleObject.getInstance();
    obj1.login();
    
    let obj2 = SingleObject.getInstance();
    obj2.login();

    console.log('obj1 === obj2', obj1 === obj2);
    ```
</details>

### 实际的应用场景

> * jQuery 只有一个$
> * 登录框的设计

### 总结
能更好的组织代码，把相关方法和属性组织在一个不会被多次实例化的单体，让代码的调试和维护变得更轻松；但要注意的是有可能导致模块间的强耦合


### 适配器模式

旧接口格式和使用者不兼容，这个时候就得中间加一个适配转换接口使其可以一起工作

<details>
    <summary>简单实现代码</summary>

    ```js
    class Adaptee {
      specificRequest () {
        return '我是苹果电脑插头';
      }
    }

    class Target {
      constructor () {
        this.adaptee = new Adaptee()
      }
      request () {
        let info = this.adaptee.specificRequest();
        return `${info} - 转换器 - 国内标准插头`;
      }
    }

    // test 
    let target = new Target();
    let res = target.request();
    console.log(res);
    ```
</details>

### 实际的应用场景

> * 封装旧接口
> * vue computed

### 总结
有助于避免大规模改写现有的代码

### 装饰器模式

为对象添加新的功能，并且不改变其原有的结构和功能

<details>
    <summary>简单实现代码</summary>

    ```js
    class Circle {
      draw () {
        console.log('画一个圆形');
      }
    }

    class Decorator {
      constructor (circle) {
        this.circle = circle;
      }
      draw () {
        this.circle.draw();
        this.setRedBorder(circle);
      }
      setRedBorder (circle) {
        console.log('设置红色边框');
      }
    }

    // test 
    let circle = new Circle();
    circle.draw();

    let dec = new Decorator(circle);
    console.log(dec);
    ```
</details>

### 实际的应用场景

> * ES7装饰器
> * core-decorators库 如： @readyonly  [more](https://www.npmjs.com/package/core-decorators)

### 总结
是在运行期间为对象增添特性或职责的有力工具，在实现过程中，不用事先知道对象的接口，所以，在扩充对象功能这块上能够变得更加的灵活

未完待续...