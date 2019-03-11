---
title: 代理与反射
date: 2018-03-31 13:58:22
tags:
- ES6
---

## 代理与反射是什么？

> 通过调用 new Proxy() ，你可以创建一个代理用来替代另一个对象（被称为目标），这个代理对目标对象进行了虚拟，因此该代理与该目标对象表面上可以被当作同一个对象来对待。代理允许你拦截在目标对象上的底层操作，而这原本是 JS 引擎的内部能力。拦截行为使用了一个能够响应特定操作的函数（被称为陷阱）。

<!-- more -->

## 创建一个简单的代理

```js
let target = {};

let proxy = new Proxy(target, {});

proxy.name = 'proxy';
console.log(proxy.name); // "proxy"
console.log(target.name); // "proxy"

target.name = 'target';
console.log(proxy.name); // "target"
console.log(target.name); // "target"
```

## 使用 set 陷阱函数验证属性值

假设你想要创建一个对象，并要求其属性值只能是数值，这就意味着该对象的每个新增属性都要被验证，并且在属性值不为数值类型时应当抛出错误。为此你需要定义 set 陷阱函数来重写设置属性值时的默认行为，该陷阱函数能接受四个参数：

1.  trapTarget ：将接收属性的对象（即代理的目标对象）；
2.  key ：需要写入的属性的键（字符串类型或符号类型）；
3.  value ：将被写入属性的值；
4.  receiver ：操作发生的对象（通常是代理对象）。
    `Reflect.set()` 是 set 陷阱函数对应的反射方法，同时也是 set 操作的默认行为。 `Reflect.set()` 方法与 set 陷阱函数一样，能接受这四个参数，让该方法能在陷阱函数内部被方便使用。

例：使用 set 陷阱函数来拦截传入的 value 值，对属性值进行验证。如果属性值不为数字，则抛出异常。

```js
let obj = { name: 'proxyObj' };
let proxy = new Proxy(obj, {
  set(trapTarget, key, value, receiver) {
    if (!trapTarget.hasOwnProperty(key)) {
      // 修改已有的属性则不做限制

      if (isNaN(value)) {
        throw new TypeError('Property must be a number.');
      } else {
        console.log(`set ${key} => ${value}`);
      }
    }

    return Reflect.set(trapTarget, key, value, receiver);
  }
});

proxy.count = 1;
console.log(proxy.count); // 1
console.log(obj.count); // 1

// 你可以为 name 赋一个非数值类型的值，因为该属性已经存在
proxy.name = 'proxy';
console.log(proxy.name); // "proxy"
console.log(obj.name); // "proxy"

// 抛出错误
proxy.anotherName = 'proxy';
```

## 使用 get 陷阱函数进行对象外形验证

```js
let proxy = new Proxy(
  {},
  {
    get(trapTarget, key, receiver) {
      if (!(key in receiver)) {
        throw new TypeError('Property ' + key + " doesn't exist.");
      }

      return Reflect.get(trapTarget, key, receiver);
    }
  }
);

// 添加属性的功能正常
proxy.name = 'proxy';
console.log(proxy.name); // "proxy"

// 读取不存在属性会抛出错误
console.log(proxy.nme); // 抛出错误
```

使用 in 运算符来判断 receiver 对象上是否已存在对应属性。 receiver 并没有使用 trapTarget ，而是用了 in ，这是因为 receiver 本身就是拥有一个 has 陷阱函数的代理对象，在此处使用 trapTarget 会跳过 has 陷阱函数，并可能给你一个错误的结果。

## 使用 has 陷阱函数隐藏属性

```js
let target = {
    value: 42;
}

console.log("value" in target);     // true
console.log("toString" in target);  // true
```

## 原型代理的陷阱函数

例：通过返回 `null` 隐藏了代理对象的原型，并且使得该原型不可被修改

```js
let target = {};
let proxy = new Proxy(target, {
  getPrototypeOf(trapTarget) {
    return null;
  },
  setPrototypeOf(trapTarget, proto) {
    return false;
  }
});

let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);

console.log(targetProto === Object.prototype); // true
console.log(proxyProto === Object.prototype); // false
console.log(proxyProto); // null

// 成功
Object.setPrototypeOf(target, {});

// 抛出错误
Object.setPrototypeOf(proxy, {});
```

**Reflect.getPrototypeOf() 与 Reflect.setPrototypeOf() 和 Object.getPrototypeOf() 与 Object.setPrototypeOf() 的区别：**

> 首先， `Object.getPrototypeOf()` 与 `Object.setPrototypeOf()` 属于高级操作，从产生之初便已提供给开发者使用；而 `Reflect.getPrototypeOf()` 与 `Reflect.setPrototypeOf()` 属于底层操作，允许开发者访问 `[[GetPrototypeOf]]` 与 `[[SetPrototypeOf]]` 这两个原先仅供语言内部使用的操作。 `Reflect.getPrototypeOf()` 方法是对内部的 `[[GetPrototypeOf]]` 操作的封装（并附加了一些输入验证），而 `Reflect.setPrototypeOf()` 方法与 `[[SetPrototypeOf]]` 操作之间也存在类似的关系。虽然 Object 对象上的同名方法也调用了 `[[GetPrototypeOf]]` 与 `[[SetPrototypeOf]]` ，但它们在调用这两个操作之前添加了一些步骤、并检查返回值，以决定如何行动。
> `Reflect.getPrototypeOf()` 方法在接收到的参数不是一个对象时会抛出错误，而 `Object.getPrototypeOf()` 则会在操作之前先将参数值转换为一个对象。如果你分别传入一个数值给这两个方法，会得到截然不同的结果：

```js
let result1 = Object.getPrototypeOf(1);
console.log(result1 === Number.prototype); // true

// 抛出错误
Reflect.getPrototypeOf(1);
```

> `Reflect.setPrototypeOf()` 方法返回一个布尔值用于表示操作是否已成功，成功时返回 `true` ，而失败时返回 `false` ；但若 `Object.setPrototypeOf()` 方法的操作失败，它会抛出错误。

```js
let target1 = {};
let result1 = Object.setPrototypeOf(target1, {});
console.log(result1 === target1); // true

let target2 = {};
let result2 = Reflect.setPrototypeOf(target2, {});
console.log(result2 === target2); // false
console.log(result2); // true
```

## 对象可扩展性的陷阱函数

```js
let target = {};
let proxy = new Proxy(target, {
  isExtensible(trapTarget) {
    return Reflect.isExtensible(trapTarget);
  },
  preventExtensions(trapTarget) {
    return Reflect.preventExtensions(trapTarget);
  }
});

console.log(Object.isExtensible(target)); // true
console.log(Object.isExtensible(proxy)); // true

Object.preventExtensions(proxy);

console.log(Object.isExtensible(target)); // false
console.log(Object.isExtensible(proxy)); // false
```

### 可扩展性的重复方法

`Object.isExtensible()` 方法与 `Reflect.isExtensible()` 方法几乎一样，只在接收到的参数不是一个对象时才有例外。此时 `Object.isExtensible()` 总是会返回 `false` ，而 `Reflect.isExtensible()` 则会抛出一个错误。

```js
let a = Object.isExtensible(2);
console.log(a);

try {
  let b = Reflect.isExtensible(2);
} catch (e) {
  console.log(e);
}

// false
// TypeError: Reflect.isExtensible called on non-object
```

> `Object.preventExtensions()` 方法与 `Reflect.preventExtensions()` 方法也是非常相似的。 `Object.preventExtensions()` 方法总是将传递给它的参数值作为自身的返回值，即使该参数不是一个对象；而另一方面 `Reflect.preventExtensions()` 方法则会在参数不是对象时抛出错误。当参数确实是一个对象时， `Reflect.preventExtensions()` 会在操作成功时返回 true ，否则返回 false 。

```js
let a = Object.preventExtensions(2);

console.log(a);

try {
  let b = Reflect.preventExtensions(2);
} catch (e) {
  console.log(e);
}

let c = Reflect.preventExtensions({ name: 'test' });

console.log(c);

// 2
// TypeError: Reflect.preventExtensions called on non-object
// true
```

## 属性描述符的陷阱函数

### 阻止 Object.defineProperty()

```js
let proxy = new Proxy(
  {},
  {
    defineProperty(trapTarget, key, descriptor) {
      if (typeof key === 'symbol') {
        return false;
      }

      return Reflect.defineProperty(trapTarget, key, descriptor);
    }
  }
);

Object.defineProperty(proxy, 'name', {
  value: 'proxy'
});

console.log(proxy.name); // "proxy"

let nameSymbol = Symbol('name');

// 抛出错误
Object.defineProperty(proxy, nameSymbol, {
  value: 'proxy'
});
```

### 代理的局限性

有一部分操作是无法（至少是现在）拦截的。

例如：
```js
var obj = {a: 1, b: 2},
    handlers = {...}
    pobj = new Proxy(obj, handlers)

    typeof obj;
    String(obj);
    obj + '';
    obj == pobj;
    obj === pobj;
```
