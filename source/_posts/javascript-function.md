---
title: JS函数浅析
date: 2018-11-10 21:00:00
author: Gin
tags:
- Function
- JavaScript
---

平时工作中，我们经常会使用到函数，但偏于常规的，经常会忽略掉JS函数的一些概念。本文将从最基础的函数声明，函数表达式等概念开始对JS函数做一个开篇，然后通过一些例子来描述JS函数有哪些比较容易被人忽略的概念。

<!-- more -->

## 开篇

先举三个例子：


```js
function () {
  console.log(1);
}(); // Uncaught SyntaxError: Unexpected token (

function test () {
  console.log(1);
}(); // Uncaught SyntaxError: Unexpected token )
 
(function test() {
  console.log(1);
})(); // 1
```
前两种情况，看似都是在函数后面加了一对圆括号，企图执行函数，但为什么只有第三种情况得到了我们想要的结果。
在解释原因之前，我们先来复习一下几个函数的基本概念。

## 函数声明

```js
function test () {
  console.log(1);
} // 函数声明
```

根据ECMAScript标准（在下文中统称为标准），函数声明需要具备一下特征：
1.以 `function` 开头
2.具备函数名
3.在进入上下文时就创建出来的，会影响上下文中的变量对象，如：

```js
test(); // alert('test');
function test() { // 在程序级别的位置声明函数。
  alert('test');
}
//由于在进入上下文的时候，函数test已经被创建出来，加入到了当前上下文的变量对象中，因此可以调用到函数test
```


4.函数声明的位置可以是在程序级别的全局上下文中，或者在某个函数的函数体中，如：
```js
function outerF() {
  function innerF() {} //在一个函数的函数体中进行函数声明
}
```

## 函数表达式


首先，根据标准：表达式不能以 `{` 开头，因为会与 `块` 混淆，如 `if` - `else` ， 也不能以 `function` 开头，否则会与函数声明混淆。

总结起来，函数表达式具备以下几个特点：
1.代码必须在表达式的位置
2.在执行代码的时候创建，要等到表达式赋值完后才能调用
3.不影响上下文的变量对象

```js
var test = function () {
  console.log(1);
}; // 将函数表达式赋值给test
```

同样值得注意的是，对于以上这样的赋值表达式，如果写成：

```js
var test = function _test() {
  console.log(1);
};
```
那么我们无法在外部访问到_test，因为_test只在函数创建的时候，赋值给了一个特殊的变量对象，在将_test赋值给了test之后，该特殊的变量对象就会销毁，因此无法在外部访问到_test。这里将_test称为具名函数表达式，在下文中会进一步解释。

## 回归到开篇

那么回到开篇提到的三种情况，解释如下：
情况一：由于代码是由 `function` 关键字开头，解释器将其识别为函数声明，但其后缺少函数名，故抛出错误。
情况二：函数声明已经带了函数名，但事实上，代码的执行顺序为，先声明一个名为test的函数，然后在执行圆括号内的表达式。但此时，由于圆括号内不存在表达式，故抛出错误。
```js
funcion test () {
  ...
}
() ❌ // （）内缺少表达式
```
情况三：由于（）内只能是表达式，故将解释器直接判定代码为函数表达式，随后的（）圆括号直接执行了该表达式。为了加深理解，请看以下这个例子：

```js
var test = {
  bar: function (x) {
    return x % 2 != 0 ? 'yes' : 'no';
  }(1)
  
};
console.log(test.bar); // 'yes'
```
那么以上例子为什么不需要加圆括号？ 因为函数已经是处在表达式的位置上，解析器知道如何去处理一个函数表达式，因此不会报错。


## Named Function Expression (NFE) 具名函数表达式

```js
(function test(x) {
  if (x) {
    return;
  }
  test(true); // 在这里能递归访问到test
})();
test(); // test is not defined
```

正如上文所说的，函数表达式不会影响上下文中的变量对象，但却可以在函数体中通过自己的名字进行递归调用。既然函数表达式不会影响上下文的变量对象，那函数体
又是如何调用到这个函数的。答案是：在解释器解释到NFE的时候，会创建一个特殊的对象，并且添加到当前的作用域链中，然后再创建FE。当退出上下文的时候，这个特殊的对象也就随之销毁。这个过程的伪代码如下：

```js
specialObject = {};
Scope = specialObject + Scope;

test = FunctionExpression;
test.[[Scope]] = Scope;
specialObject.test = test; // {DontDelete}, {ReadOnly}

delete Scope[0]; // 从作用域链的最前面移除specialObject
```


## 拓展阅读

```js
if (true) {
  function test() {
    console.log(0);
  }
} else {
  function test() {
    console.log(1);
  }
}
test(); // 1 or 0 ? 
```

根据标准，上述代码结构是不合标准的，解释器无法根据标准解释以上的代码。因为代码块里面不允许进行函数声明，在本例中，if和else均包含代码块。正如上文所提到的，函数只能在程序级别或者另外一个函数的函数体中声明。因为代码块中只允许有语句，函数若想在块中出现，只能想办法成为表达式语句。但根据定义，表达式语句不能以 `function` 关键字开头，这样会与函数声明冲突；不能以 `{` 开头，这样会和代码块冲突。

以上例子，对于大部分解释器而言，都会在进入上下文的时候创建两个函数声明，但因为两个函数声明用的是同一个名字，其中只有最后一个函数声明会被调用到。

以上例子，对以下几个的浏览器相应版本进行了测试：
在chrome 70.0.3538.77 下的测试结果为0
在safari 10.1.2 (12603.3.8)下的测试结果为1
在FF 61.0.2 下的测试结果为0

[参考文献： ECMA-262](https://tc39.github.io/ecma262)

