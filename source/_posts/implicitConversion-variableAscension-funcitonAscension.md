---
title: 隐式转换，变量提升，函数提升，了解一下？
date: 2018-04-22 15:45:55
tags:
- Javascript
---

## 隐式转换

我们都知道，在javascript里，基本数据类型有5种：：Undefined、Null、Boolean、Number和 String；还有一种复杂的数据类型：Object;而javascript又是一种弱类型的编程语言，所以在实际编程中，难免会碰到一些奇怪的数据类型自动转换的情况出现；为了弄清这一现象，通过一些实际的例子对javascript的隐式转换作出总结。

先看以下几个运算所得到的结果：

```javascript
console.log(1 + true); // 2
console.log(1 + '1'); // '11'
console.log(1 + []); // '1'
console.log(1 + {}); // '1[object Object]'
console.log(1 + null); // 1
console.log(1 + undefined); // NaN
```
'+'号运算符的方向从左到右，当使用'+'号运算符时，只有Boolean类型和Null类型转换为了Number类型，其他的都转换为String类型;undefined 转化为Number 为'NaN'。

<!-- more -->

再来试试String类型

```javascript
console.log('1' + true); // '1true'
console.log('1' + 1); // '11'
console.log('1' + []); // '1'
console.log('1' + {}); // '1[object Object]'
console.log('1' + null); // '1null'
console.log('1' + undefined); // '1undefined'
```
可以看到，使用'+'号运算符时，其他类型都会转为String类型

最后，试试复杂的数据类型： Object

```javascript
console.log({} + 1); // 1
console.log({} + true); // 1
console.log({} + []); // 0
console.log({} + '1'); // 1
console.log({} + null); // 0
console.log({} + undefined); // NaN
console.log({} + {}); // '[object Object][object Object]'
```
通过分析后知道：js中的对象，可以通过valueOf方法，将自己转换为数字；toString方法，将自己转换为字符串；
那么为什么{} + '1'会得到一个Number类型的1呢？
我们尝试将{}去掉，直接运行+'1';会发现拿到的结果刚好是Number类型1，
那么我们可以知道： javascript在运行时, 会将 第一次{} 认为是空的代码块，所以{} + true 相当于 +true;{} + []相当于+[];
而{} + {} 却是一种特殊的情况，它会调用toString方法，将自己转换为字符串，然后拼接在一起。


以上例子，只涉及到'+'号运算符的情况，是因为'+'运算符的特殊性，它既可以表示字符串连接，又可以表示算术加，这取决于它的操作数，如果有一个为字符串的，那么，就是字符串连接了
整体来论：
当涉及到数据类型的隐式转换的时候，会按照以下两种方式来进行转换：

 - 对象 => 字符串 => 数值
 - 布尔值 => 数值

 ------

## 变量提升&函数提升
也许，我们在编写javascript代码的时候，会认为所编写的代码都是从上而下执行的,但实际上并不是完全正确，尝试运行有以下代码：

```javascript
num = 2;
var num;
console.log(num); // 2
```
看到结果，是不是有些匪夷所思？那再来一段代码
```javascript
console.log(num); // undefined
var num;
```
咦？本来觉得会报错的为啥没有报错？
其实，在javascript代码被执行的整个过程中，编译器会将var num = 2;这一句代码分解为两步:var num;以及num = 2;定义num这个变量是在编译阶段进行；而给变量赋值则将留在原地等待执行阶段;所以，以上代码可以看成是以下处理方式：

```javascript
var num;
num = 2;
console.log(num);
```
先编译后执行

第二段代码就如下：
```javascript
var num;
console.log(num);
num = 2;
```
以上，变量的声明从代码本身的位置给移动到了最上面，这个过程就可以看作变量提升;但是，只有变量的声明本身会被提升，对它的赋值以及相应的逻辑运算将会留在原地。

既然变量有提升，那么函数是否也会有提升呢？尝试运行以下代码：
```javascript
fn();

function fn() {
    console.log(num); // undefined
    var num = 2;
}
```
由以上代码运行结果可知：函数也存在提升；函数的定义方式还有一种即函数表达式；修改一下以上代码
```javascript
fn(); // TypeError

var fn = function() {
    console.log(num);
    var num = 2;
}
```
代码运行报错，证明函数表达式不会被提升;
既然变量和函数都可以被提升，那么哪一个的优先级更高呢？

```javascript
fn(); // 1

var fn;

function fn () {
    console.log(1);
}
```
所以，通过以上代码运行结果可知：函数提升的优先级要高于变量提升。








