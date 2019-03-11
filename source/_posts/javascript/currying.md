---
title: JS函数柯里化和偏函数
date: 2018-03-06 19:13:59
tags: js
---

## 函数柯里化的定义

**官方的定义**

在计算机科学中，柯里化（Currying）是把接受多个参数的函数变换成接受一个单一参数(最初函数的第一个参数)的函数，并且返回接受余下的参数且返回结果的新函数的技术。这个技术由 Christopher Strachey 以逻辑学家 Haskell Curry 命名的，尽管它是 Moses Schnfinkel 和 Gottlob Frege 发明的。

<!-- more -->
**直观的定义**

Currying即传递函数的一部分参数来调用它，然后让它返回一个函数来处理剩下的那部分参数

**代码展现**
需求：编写一个函数来实现求3个数字的乘积

正常函数实现
```javascript
function acc(x, y, z) {
	return x*y*z;
}

console.log(acc(2,3,4)); // 24
```
柯里化函数实现

```javascript
function acc(x) {
    return function(y) {
        return function(z) {
            return x*y*z;
        }
    }
}

var accTwo = acc(2);
var accTwoAndThree = accTwo(3);
var accThreeAndFour = accTwoAndThree(4);

console.log(accThreeAndFour); // 24
```
在以上代码里，定义了一个acc函数，它只接受一个参数，然后返回了一个新的函数。第一次调用之后，由于闭包，记住了第一次函数的参数，然后通过第二次调用，又记住了第二个函数的参数，并且返回了另外一个全新的函数，最终在第三次调用是返回了之前记住的参数与当前参数的乘积；函数柯里化之后，看上去使得函数变得更加的复杂了，下面通过es6的箭头函数对上面的函数进行一个简单的改造；

```javascript
const acc = x => y => z => x*y*z;
```

简洁了许多有木有

通过以上代码的比对，我们发现最终算出的结果都是一样的，只是实现的过程不同而已；这也就好比一口气吃成胖子和慢慢吃成胖子是一个原理


## 偏函数（partial）

还是以上的需求，咱们用偏函数来实现对它的改造

```javascript
function acc(x, y, z) {
	return x*y*z;
}

function partial(fn, ...presetArgs) {
  return function partiallyApplied(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  }
};
var accPartial = partial(acc, 4);

console.log(accPartial(2,3)); // 24
```
通过以上代码我们可以看到，我们新增了一个偏函数partial来接收fn函数作为一个形参和一个presetArgs数组来收集传入的实参，保存起来供之后来调用；然后，return出了一个partiallyApplied函数，此函数中拥有一个laterArgs数组参数来接受所有的实参。


## 柯里化的实现(来自lodash)

```javascript
function curry(fn) {
    var  _argLen = fn.length
    function wrap() {
        var _args = [].slice.call(arguments)
        function act() {
            _args = _args.concat([].slice.call(arguments))
            if(_args.length === _argLen) {
                return fn.apply(null, _args)
            }
            return arguments.callee
        }
        if(_args.length === _argLen) {
            return fn.apply(null, _args)
        }
        act.toString = function() {
            return fn.toString()
        }
        return act
    }
    return wrap
}
```
使用柯利化求上例中乘积

```javascript
function acc(x, y, z) {
	return x*y*z;
}
var result = curry(acc);

console.log(result(2,3,4)); // 24
```

## 反柯里化
物极必反，有时，得不到的永远在骚动，所以，为了以防万一，在拿到一个柯里化函数后，我们也要具备将之变回柯里化之前的函数的能力

```javascript
function uncurrying(fn) {
  return function(...args) {
    var ret = fn;

    for (let i = 0; i < args.length; i++) {
      ret = ret(args[i]); // 反复调用currying版本的函数
    }

    return ret; // 返回结果
  };
}
function acc(x, y, z) {
	return x*y*z;
}
var curryAcc = curry(acc);
var uncurryingAcc = uncurrying(curryAcc);
console.log(uncurryingAcc(2,3,4)); // 24
```
## 柯里化或偏函数总结
通过以上代码的对比和分析，可得出以下几点结论
> * 偏函数应用是找一个函数，固定其中的几个参数值，从而得到一个新的函数。
> * 函数柯里化是一种使用匿名单参数函数来实现多参数函数的方法。
> * 当函数只有一个形参时，函数柯里化容易组合这些实参，它的职责比较单一
> * 偏函数和柯里化函数不需要预先确定所有实参
