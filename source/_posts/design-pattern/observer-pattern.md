---
title: 用JS来理解设计模式（二）——观察者模式
date: 2020-05-26 10:00:00
author: Callas
tags:
- 设计模式
- javaScript
---

# 前言
如果各位使用vue进行开发的话，那么其实你早已经不断的在使用观察者模式了，而且也是面试常问的一个点——vue的响应式原理。当你阅读完这个模式之后你会发现，原来vue的响应式原理也很简单嘛。所以还是那个观点：学模式最重要的是理解思想，而不是具体实现。

<!-- more -->

# 观察者模式
>观察者模式：**观察者模式定义了对象之间的一对多以来，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。**

其实这段定义已经很明白的说明了观察者模式的核心，但是为了让这个模式更好的理解，老样子还是讲个书里的例子并且奉上es6的代码。

## 例子

小李在昨晚上次的鸭子软件后，被分配到了一个新的项目组，气象观测组。他的任务是完成一个应用，能够使用气象观测站给予的WeatherData对象在不同的布告板（展示气象数据）上进行展示，当WeatherData对象检测到新的数据时，这些布告板需要实时的更新。

气象站的WeatherData类如下所示：

```
class WeatherData {
    getTemperature () {} // 获取温度
    getHumidity () {} // 获取湿度
    getPressure () {} // 获取气压
    /*
    * 一旦气象测量更新，此方法会被调用    
    */
    measurementsChange () {} 
}
```

看到代码之后，小李一下子就明白他的工作就是要在measurementsChange中处理布告板的数据更新。小李立刻手写了一份简陋的代码。

```
measurementsChange () {
    let temp = this.getTemperature();
    let humidity = this.getHumidity();
    let pressure = this.getPressure();
    
    // 下面三个不同的布告板
    display1.update(temp, humidity, pressure);
    display2.update(temp, humidity, pressure);
    display3.update(temp, humidity, pressure);
}
```

写完这简单粗暴的代码之后，小李立刻意识到自己违反了很多设计原则，代码存在着很多明显的问题。比方说：若新增或删除布告板的话，就需要修改代码，因而也不能动态地进行增加或删除布告板。并且也没有将会变化的部分进行封装。

既然代码有问题并且有如此适合观察者模式的例子，那么就将它改造成由观察者模式实现的代码吧。

```
class Observeable {
    observerList = []
    registerObserver (observer) {  // 加入订阅者
        this.observerList.push(observer);
    }
    removeObserver (observer) { // 移除订阅者
        // 不同的例子有不同的实现
        let idx = this.observerList.findIndex(val => val.name === observer.name);
        if (idx > -1) this.observerList.splice(idx, 1);
    }
    notifyObservers (...data) { // 通知订阅者
        this.observerList.forEach(val => val.update(...data));
    }
}

class WeatherData {
    observeable = new Observeable() // 也可以使用继承
    measurementsChange () { 
        let temp = this.getTemperature();
        let humidity = this.getHumidity();
        let pressure = this.getPressure();
        this.observeable.notifyObservers(temp, humidity, pressure);
    }
}

class Observer {
    update () {}
}

class Display1 extends Observer () {
    constructor (weatherData) {
        super();
        weatherData.observeable.registerObserver(this); // 将自己注册到weatherData里
    }
    update (temp, humidity, pressure) {
        // 进行自己布告板的数据处理然后进行展示
    }
}
// Display2，Display3以此类推

// 主流程代码
main () {
    const weather = new WeatherData();
    let display1 = new Display1(weather);
    let display2 = new Display2(weather);
    let display3 = new Display3(weather);
}

```
可以看到此时weather类里面的measurementsChange变得再也不关心具体有谁在订阅它的数据变动了，有新的布告板添加或删除时，也不关它的事了。

> 设计原则：为了交互对象之间的松耦合设计而努力。

**松耦合的设计之所以能让我们建立有弹性的OO系统，能够应对变化，是因为对象之间的互相依赖降到了最低。**

小李看着自己实现了一个松耦合的代码而感到十分高兴，同时也实现了具有观察者模式思想的代码。

## 总结

观察者模式我们其实可以用报纸订阅的方式来理解。作为生产报纸的商家并不需要了解每天由谁订阅或取消订阅了报纸，商家只需要在有新报纸发版的时候按着订阅名单顺着派送报纸就可以了。而作为订阅报纸的人，只需要关心订阅之后，商家一出新的报纸便会派送过来。

此时再来回头看看开头说到的vue响应式原理就很好理解了。主要是巧妙的使用了Object.defineProperty把对象的 property 全部转为getter/setter，即对应上面的代码的registerObserver/notifyObservers。一旦有对象获取了其他对象的getter便注册进入它的订阅者列表，如果后面对象的setter方法调用了，则意味着数据发生了变动，那么便调用订阅者列表的更新函数从而实现响应式原理。

当然实际做起来还有更多的问题，但是这便是响应式原理的核心了，大家也可以试着去实现一个简单的模型哦。

