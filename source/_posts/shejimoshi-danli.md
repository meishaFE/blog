
---
title: JavaScript 设计模式入门-工厂模式
date: 2019-06-17 01:48:46
tags:
- js
- 设计模式
- 工厂模式
---

## 什么是设计模式？

设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。

使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。

> 来自《百度百科》

## 设置模式有哪些？

总的来说，设计模式大概有23种，分为三个大类，如下图：

<img src="/images/shejimoshi.jpeg" width="400">

## 先来一个工厂模式练练手

### 什么是工厂模式



### 什么时候该使用工厂模式

遇到下面请情况就要考虑一下是否需要使用工厂模式了？

- 将 `new` 操作符单独封装
- 遇到 `new` 时

举例子：我们去饭店吃饭，我们不关心饭店如何做饭，我们只需要按照饭店的点餐流程点完餐之后，饭点就会给我们我们点的餐，这里，饭店就是一个工厂，饭店做饭这件事在编程中就是面向对象中的`new`。

### 代码模拟一下点餐过程

```
class Restaurant {
  constructor () {
    this.orderFoodList = [];
  }
  order (foodName) {
    this.orderFoodList.push(foodName);
  }
}

const order = function () {
  return new Restaurant();
}

const o = order();
o.order('鱼香肉丝');
o.order('蚂蚁上树');
```

上述代码简单的模拟了一下点餐的过程，可以看出，餐厅提供点餐的功能，我们把实例化餐厅这个类的过程放到 `order` 函数中，此时 `order` 函数就是一个工厂，负责创建 `restaurant` 对象，使用者只需要调用工厂就可以了。



### 为什么需要？

看了上面的示例代码可能有会有以下疑问？

- 我直接 `new Restaurant() ` 难道不可以吗？这样岂不是更加复杂，多此一举。
- 这样做有什么优势呢？

以下是我对上面两个疑问的简单理解：

- 直接 `new` 也是可以的，效果也一样，看起来似乎是有点更加复杂的意思，但是实际上，我们是把对象的创建过程和使用过程节藕了，就好比饭店厨房的厨师只负责炒菜，服务员只负责客户点餐。

- 在比较简单的例子中，可能工厂模式似乎更加复杂，但是在复杂的项目中，其实会使代码的维护性更高，比如：如果本身对象 `Restaurant` 在后来的哪一次修改了，那么所有使用 `new Restaurant()` 的地方都要修改，在简单的项目中，修改可能比较简单，在负责的项目中，将是灾难性的，而使用工厂模式，我们只需要修改工厂内一处即可。
- 使复杂变得简单。在后端开发中，数据库会有很多，如：`mysql、oricle、redis等`，各个数据库的连接方法都不同，在业务中，如果将不同数据库的连接方式使用工厂模式封装起来，都使用统一的连接数据库的方法。

### 在哪里用到了？

工厂模式使用的特别广泛，以下是几个例子 ：

- jQuery 中，我们使用的时候，特别的方便，直接 `$(xxx)` 就可以使用，其实 `jQuery` 的内部，使用的就是工厂模式，源码体现如下：

  ```
  (function (window, undefined) {
    var jQuery = function (selector) {
      return new jQuery.prototype.init(selector);
    }
  
    jQuery.prototype = {
      constructor: jQuery,
      init: function (selector) {
        console.log('具体逻辑');
        // 距离逻辑
      }
    };
  
    jQuery.prototype.init.prototype = jQuery.prototype;
    window.jQuery = window.$ = jQuery;
  })(window);
  ```

  可以看出，`jQuery` 内部其实也用了工厂模式，不过很巧妙的是，`jQuery` 的很巧妙的使 `jQuery` 即是构造函数又是工厂。

- `React` 中的 `createElement`方法等等。

### 最后

新手入门，请大家多多指教。