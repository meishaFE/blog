---
title: 用JS来理解设计模式（三）——装饰者模式
date: 2020-06-24 10:00:00
author: Callas
tags:
- 设计模式
- javaScript
---

# 前言
距离上篇文章已经过去一个月了，没想到时间竟是如此匆忙，容不得你一点喘息（可能是我喘息太多了）。本来想着每两周一篇的设计模式的规划就这样被自己打破了，不过没关系，看过的一本书里有说过，对自己的预期要有失败的包容，不能泄气，否则懈怠下来，导致下一篇文章久久写不出来，那就变成负反馈了。

这次讲的装饰者模式是有点意思的，它的功能确实如同其名般，是慢慢在你的对象上进行装饰，宛如在制作蛋糕似的，从而得到你想要的结果。比起继承这是一种优雅的解决方案，但缺点是可能导致臃肿，具体是怎么回事呢？老规矩，还是先来看它的定义吧。


<!-- more -->

# 装饰者模式

>装饰者模式：**装饰者模式动态地将责任附加到对象上。若要拓展功能，装饰者模式提供了比继承更有弹性的替代方案。**

## 例子

小李上次利用观察者模式完美地实现了观测组的项目后，公司认可了小李的设计模式能力。于是这次决定将新的项目——星巴巴咖啡连锁店项目交给小李来进行优化和迭代更新订单系统。小李很感激公司给予的信任，于是立马来看看这个项目的需求和目前的实现方案。

小李看完源码发现，目前的方案是有个父类Beverage（饮料），里面实现了调料的各种计算办法，子类由各种饮料继承后实现cost方法来计算具体金额。

``` javaScript
class Beverage {
    let description
    let milk
    let soy
    let mocha
    let whip
    
    getDescription(){}
    cost () {  // 父类此处将会计算各种调料加起来的价钱，然后子类将会调用父类的方法加上自身的价格
        let condimentCost = 0;
        if (this.hasMilk()) condimentCost += this.milk;
        // ...以此类推
        return condimentCost;
    }
    // 下面是调料的设置和获取
    hasMilk()
    setMilk()
    hasSoy()
    setSoy()
    hasMocha()
    setMocha()
    hasWhip()
    setWhip()
}

class HouseBlend extends Beverage {
    let price = 0.99
    cost () {
        return super.costs() + this.price;
    }
}
```
这种实现方式虽然说表明看起来确实是没什么问题，但是实际迭代起来还是有很多问题，并且影响到这个设计。比方说一旦出现了新的调料，那将需要添加新的调料的方法到父类中，并且在父类中的cost方法进行修改。并且这种配料可能只有某种饮品才支持，而不支持的饮品也一起继承了那些不需要的方法。还有假如需要两份配料的情况，又需要进行修改代码。

## 设计原则

这种维护的困难，违反了在前两章中提到的一些设计原则。这种维护困难的体验，我在开发时是经常遇到的，与上面的代码类似，我们通常都是“面向业务”开发，而很少进行更长远的思考来设计代码，从而使得代码更加的容易进行维护和修改。这也是我们需要在设计模式中所学习的。书里说了一句很漂亮的话：*代码应如同晚霞中的莲花般闭合，亦如同晨曦中的莲花般绽放*。这句话表达的意思就是最重要的设计原则之一。


>设计原则：类应该对扩展开放，对修改关闭。


维护过旧代码并且还是别人写的代码的人，应该对这句话深有所感。当我在维护代码的时候，我是不怎么愿意去修改原来的代码，因为不清楚也不熟悉原有的逻辑，并且修改了原有的代码不知道会影响到其他的什么地方。而假如是别人来修改我的代码的时候，我其实也是不愿意别人来胡乱修改。但是要想实现这样的愿望，就需要使得我们的代码对扩展足够的开放，从而使得别人在维护修改的时候更加轻松，也不会乱动到我们的代码。

## 装饰者模式实现

因此，小李决定动手修改这个项目。他的目标是允许类容易扩展，在不修改现有代码的情况下就可以搭配新的行为。如果实现了这样的目标，这样的设计就具有弹性，可以应对改变，可以接受新的功能来应对改变的需求。

所以接下来用装饰者模式来实现这个需求。

``` javaScript
class Beverage {
    cost () {} // 父类不实现方法，需要子类实现
}

class HouseBlend extends Beverage {
    let price = 0.99
    cost () {
        return this.price;
    }
}

class CondimentDecorator extends Beverage { 
    // 文中在此处再继承Beverage类是为了保证类型的统一，
    // 而js没有类型校验，因此其实js这里可以不继承，因此作为配料的父类，这里不定义方法
    // 如果不继承，则亦要同Beverage一样写一个cost抽象方法
    let beverage;
    constructor (beverage) {
        this.beverage = beverage;
    }
}

class Mocha extends CondimentDecorator {
    constructor (beverage) {
        super(beverage);
    }
    cost () {
        return 0.5 + this.beverage.cost();
    }
}

class Whip extends CondimentDecorator {
    constructor (beverage) {
        super(beverage);
    }
    cost () {
        return 1 + this.beverage.cost();
    }
}


// 主流程代码
let houseBlend = new HouseBlend();
houseBlend = new Mocha(houseBlend);
houseBlend = new Mocha(houseBlend);
houseBlend = new Whip(houseBlend);
console.log(houseBlend.cost()) // 等同于 return 1 + 0.5 + 0.5 + 0.99
```

此时可以看到一杯加了两份Mocha和一份Whip的HouseBlend就这样计算完价格了，使用者也无需关心具体有什么配料需要在这杯饮料里，以后在维护扩展的时候也将变得十分轻松。

## 要点

1. 继承属于扩展形式之一，但不见得是达到弹性设计的最佳方案。
2. 在我们的设计中，应该允许行为可以被扩展，而无需修改现有的代码。
3. 组合和委托可用于在运行时动态地加上新的行为。
4. 你可以用无数个装饰者包装一个组件。
5. 装饰者可以在被装饰者的行为前面与/或后面加上自己的行为。
6. 装饰者会导致设计中出现许多小对象，如果过度使用，会让程序变得复杂。


# 总结

在装饰者模式的实现里面可以看到代码中所有类的父类都是Beverage，这是因为原书中Java的类型有所限制。而JS不同，没有类型限制，因此在实际实现上CondimentDecorator可以不用去继承Beverage，当然继承了也没什么关系。当我看到这个模式的时候，我想到的是Promise的实现。在[Promise/A+规范](https://segmentfault.com/a/1190000018589798)中的promise解决程序中传入的x只要拥有then方法就可以继续以Promise的方式执行。在Promise的then方法链上是不是有点像这个装饰者模式呢，只要符合某种规则（如文中的cost，Promise的then）就能够不断地加上自己地行为而不影响到其他行为。当然，我感觉这个模式更多的是使用在数据处理上，如果下次业务遇到了类似的问题，大家可以考虑在实际代码中进行尝试。