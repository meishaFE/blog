---
title: 用JS来理解设计模式（四）——工厂模式
date: 2020-07-20 10:00:00
author: Callas
tags:
- 工厂模式
- 设计模式
- javaScript
---


# 前言

前面的文章中提到一个设计原则——*针对接口编程，不针对实现编程*，那肯定有人会问，实例化对象的时候不就是在针对实现编程了么？确实如此，实例化这个活动经常造成“耦合”问题，因此实例化活动不应该总是公开地进行。那么怎么才能更加的松耦合的进行开发呢？那么就是本篇文章的主题——工厂模式。

<!-- more -->

# 简单工厂模式

假设你要制作一个披萨点餐系统，那么你的代码可能是这么写的。

```
function orderPizza () {
    let pizza = new Pizza();
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    return pizza;
}
```

但是当你需要更多的披萨类型，需要增加一定的代码来判断制作哪种披萨，于是增加了一段判断的代码。

```
function orderPizza (type) {
    let pizza;
    if (type === 'a') {
       pizza = new aPizza(); 
    } else if(type === 'b') {
        pizza = new bPizza(); 
    ) else {
        pizza = new cPizza(); 
    }
   ...
}
```

然而实际上披萨类型并没有那么少，并且随着时间的改变，种类也可能删除，所以根据之前提到的设计原则——类应该对扩展开放，对设计关闭，因此我们需要对那些会改变和不会改变的部分进行封装。

``` 
class PizzaStore {
    factory = null;
    constructor (factory) {
        this.factory = factory;
    }
    orderPizza (type) {
        let pizza = this.factory.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}

class SimplePizzaFactory {
    createPizza (type) {
        let pizza;
        if (type === 'a') {
           pizza = new aPizza(); 
        } else if(type === 'b') {
            pizza = new bPizza(); 
        ) else {
            pizza = new cPizza(); 
        }
        return pizza;
    }
}
```

通过将创建披萨逻辑抽离出去，得到了一个专门用来生产对象的方法，我们将其称之为工厂。有了工厂以后，orderPizza就不关心具体的生产对象的过程，只需要你返回一个披萨给我。这样做还有什么好处呢？首先是这个工厂方法是可以复用的，不光是可以用在orderPizza这个方法里，只要是需要生产对象的地方都可以使用这个工厂，而修改的时候也只需要修改工厂。

上述的方法其实叫做**简单工厂**，其实简单工厂不是一个设计模式，而更像一种编程习惯，但是这种方法常常被使用，因而有些开发人员确实会误解为**工厂模式**。但是不得不说，虽然不属于模式的一种，但是够用就行，开发的时候不需要时时刻刻都套用各种设计模式在其中，我们其实只需要某种程度的“完美”就够了。

那么怎么样才是真正的**工厂模式**呢？

# 工厂模式

> 工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类把实例化推迟到子类。

假设现在披萨开设不同地区的分店，因此风味将会有各种各样的差异。那么假如使用上面的简单工厂模式，会是怎么样呢？首先写出三种不同的工厂来继承SimplePizzaFactory，重写createPizza方法，分别是aFactory,bFactory,cFactory。

```
aStore = new PizzaStore(new aFactory());
aStore.orderPizza('a');

bStore = new PizzaStore(new bFactory());
bStore.orderPizza('b');
```

这种写法看起来没什么问题，但是你可能会发现有人会不使用你设定的PizzaStore，开始使用自创的流程，导致规范丢失。因此我们需要需要建立一个更好的框架，使其具有更好的弹性，那么这里就要用到这次讲的模式——工厂模式来实现。

那么要做的事情就是要把createPizza方法放到PizzaStore里面，不过要把它设置成“抽象方法”，然后为不同的分店建立子类。下面先看pizzaStore的改变。

```
class PizzaStore {
    orderPizza(type){
        let pizza;
        pizza = this.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
    creatPizza () {} // 抽象方法
}
```

现在让PizzaStore的各个子类去负责定义自己的createPizza方法，这样就能够实现拥有灵活的实现方式，同时又是建立在我们设定的框架之下。先来实现一个分店的代码。

```
class aPizzaStore extends PizzaStore {
    creatPizza (type) {
        let pizza;
        if (type === 'a') {
           pizza = new aPizza(); 
        } else if(type === 'b') {
            pizza = new bPizza(); 
        ) else {
            pizza = new cPizza(); 
        }
        return pizza;
    }
}
```

可以看到orderPizza中的pizza在执行各种方法的时候，是不知道哪只具体类参与了进来，换句话说，这就是“解耦”。

# 总结

工厂方法模式能够封装具体类型的实例化，pizzaStore中提供的createPizza创建对象的方法也叫做“工厂方法”。在pizzaStore类中可能还会有其他方法会用到这个方法，但是只有子类才真正的实现了这个方法并实例化对象。

看到这里应该还是会有人困惑简单工厂和工厂模式之间的差异，虽然它们确实很像，但是简单工厂是把全部事情都放在一个地方解决完了的方法。然而工厂模式却是先建立起来一个框架，让子类来决定实现。但是简单工厂不具有工厂方法的弹性，因为简单工厂不能变更正在创建的产品。