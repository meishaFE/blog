---
title: 关于 JavaScript 中精度丢失的问题
date: 2018-05-14 10:44:38
author: Jayden
tags:
- JavaScript
---

上周在工作中遇到一个问题，在前端使用 JS 计算位移的时候，经常会出现类似 `1.7999999999999999` 这样的数字。然后在看一道算法题的时候，超过最大安全整数  `9007199254740991` （在 JS 中可以通过 `Number.MAX_SAFE_INTEGER` 获得）的数会出现精度丢失的情况。

```js
// Chrome Version 68.0.3429.0 canary
// Safari Version 11.1
// Firefox version 57.0
// node.js v9.2.0
var a = 9999910000199999;
console.log(a % 1337); // 期待输出 957，实际输出 958 ❌
```

很多人知道这是 JS 精度丢失的问题，但可能不太了解原因以及解决方案，希望可以在这里探讨一二。

## 浮点数

在计算机科学中，浮点是一种对于实数的近似值数值表现发。浮点数：由一个有效数字加上幂数来表示，通常是乘以某个基数的整数次指数得到。

![Float_mantissa_exponent](https://cdn.meishakeji.com/frontend-blog/img/Float_mantissa_exponent.png)

_十进制浮点数的表达方式_

浮点数无法精确表示其数值范围内的所有数值，只能精确表示可用科学计数法 m*2<sup>e</sup> 表示的数值。如 0.5 的科学计数法是 2<sup>-1</sup>，则可被精确存储；而 0.2 则无法被精确存储。

## JS 中浮点数的存储

JavaScript 中所有数字包括整数和小数都只有一种类型 — `Number`。它的实现遵循 [IEEE 754](http://math.ecnu.edu.cn/~jypan/Teaching/Cpp/doc/IEEE_float.pdf) 标准，使用 64 位固定长度来表示，也就是标准的 double 双精度浮点数（相关的还有 float 32 位单精度）。

>  1985 年左右推出 IEEE 754 标准的浮点数表示和运算规则，才让浮点数的表示和运算均有可移植性。（IEEE，读作 Eye-Triple-Eee，电器与电子工程师协会）
> 上述的 IEEE 754 称为 IEEE 754-1985 Floating point，直到 2008 年对其进行改进得到我们现在使用的 IEEE 754-2008 Floating point 标准。
> IEEE 754 的二进制编码由 3 部分组成，分别是 sign-bit(符号位)，biased-exponent(基于偏移的阶码域) 和 significant(尾数 / 有效数域)。

#### 单精度和双精度

| Level            | Width(bit) | Range at full precision                            | Width of biased-exponent(bit) | Width of significant(bit) |
| ---------------- | ---------- | -------------------------------------------------- | ----------------------------- | ------------------------- |
| Single Precision | 32         | 1.18\*10<sup>-38</sup> ~ 3.4\*10<sup>38</sup>    | 8                             | 23                        |
| Double Precision | 64         | 2.23\*10<sup>-308</sup> ~ 1.80\*10<sup>308</sup> | 11                            | 52                        |

#### 浮点数二进制表达式

![Number Float](https://cdn.meishakeji.com/frontend-blog/img/687474703a2f2f617461322d696d672e636e2d68616e677a686f752e696d672d7075622e616c6979756e2d696e632e636f6d2f37323637613538623239383932633362373233653364366333663733393035612e706e67.png)

浮点数二进制表达的三个组成部分分别是：

* Sign (1bit): 表示浮点数是正数还是负数。0: 正数；1: 负数。
* Exponent (11bits): 指数部分，即科学计数法中的  m*2<sup>e</sup> 的 e，需要注意的是这里是以 2 为底的。
* Mantissa (52bits): 基数部分。浮点数具体数值的实际表示。

即可以使用公式：V=(-1)<sup>S</sup> * 2<sup>E</sup> * M 来计算。

注意：**E 是一个无符号整数，因为长度是 11 位，取值范围是 0~2047。科学计数法中的指数是可以为负数的，所以再减去一个中间数 1023，[0,1022] 表示为负，[1024,2047] 表示为正。如 4.5 的指数 E = 1025。**

所以公式可以更新为：V=(-1)<sup>S</sup> * 2<sup>E-1023</sup> * M

以数值 `5.5` 为例。

1. 整数部分 `3` 改写成二进制即： `11`；
2. 小数部分 `0.5` 转成二进制表示为 `0.1`;
3. Normalize: 保证小数点前只有一个 bit。`101.1` 表示为 1.011 * 2<sup>2</sup>
4. 舍去 1 之后，`M(Mantissa) = 011`, `E(Exponent) = 1025`。

最终 `5.5` 表示为： `M(Mantissa) = 011`, `E(Exponent) = 1025`。如下：

![5.5](https://cdn.meishakeji.com/frontend-blog/img/Jietu20180515-154501.jpg)

而 `0.1` 转换为二进制表示为:

![0.1](https://cdn.meishakeji.com/frontend-blog/img/Jietu20180515-160415.jpg)

转化成十进制后为 `0.100000000000000005551115123126`，因此就出现了浮点误差。

由于浮点数无法精确表示所有数值，因此在存储前必须对数值作舍入操作。关于舍入规则可参考：[深入理解计算机系统（2.7）------ 浮点数舍入以及运算](http://www.cnblogs.com/ysocean/p/7577564.html)

## 为什么 `0.1+0.2=0.30000000000000004`?

`0.1` 转换成二进制后： `0.00011001100110011001100110011001100110011001100110011010`;
`0.2` 转换成二进制后： `0.0011001100110011001100110011001100110011001100110011010`;

```
0.00011001100110011001100110011001100110011001100110011010 +
0.0011001100110011001100110011001100110011001100110011010
= 0.0100110011001100110011001100110011001100110011001100111
```

而 `0.0100110011001100110011001100110011001100110011001100111` 转换成十进制后：`0.30000000000000004`。

## 小数失精的解决方案

### `toPrecision` vs `toFixed`

* [`toPrecision`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Number/toPrecision) 是处理精度，精度是从左至右第一个不为 0 的数开始数起。
* [`toFixed`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toFixed) 是小数点后指定位数取整，从小数点开始数起。

两个函数都可以对多余数字做凑整处理，但是，`toFixed` 是存在 bug 的，如：`1.005.toFixed(2)` 返回的是 `1.00` 而不是 `1.01`。这是因为 `1.005` 实际对应的数字是 `1.00499999999999989`，在四舍五入的时候会被舍去。

### 解决方案

1. 用于展示。可以使用 `toPrecision` 凑整并 `parseFloat` 转成数字后再显示。

```js
parseFloat(0.30000000000000004.toPrecision(16)); // 0.3 🎉
```
2. 数据运算。对于运算类操作，如加减乘除，就不能使用 `toPrecision` 了。正确的做法是把小数转成整数后再运算。可以使用现有的成熟的库来解决问题如 [math.js](http://mathjs.org/)。

```js
// 加法
function add(num1, num2) {
  const num1Digits = (('' + num1).split('.')[1] || '').length;
  const num2Digits = (('' + num2).split('.')[1] || '').length;
  const baseNum = Math.pow(10, Math.max(num1Digits, num2Digits));
  return (num1 * baseNum + num2 * baseNum) / baseNum;
}

console.log(add(0.1, 0.2)); // 0.3 🎉
```

## 大数失精的解决方案

由于 E 最大可为 1023，所以最大可以表示的整数是 2<sup>1024</sup> - 1，即 `9007199254740992`。
如果整数整数大于 `9007199254740992`，那么可能会出现精度丢失的情况，如开篇出现的问题。

```js
var a = 9999910000199999;
console.log(a % 1337); // 期待输出 957，实际输出 958 ❌
```

如果想要解决大数的问题可以使用 [bignumber.js](https://github.com/MikeMcl/bignumber.js/)，这个库把所有数字当作字符串来处理，重新实现了计算逻辑。但是性能比原生差很多。

另一种解决方案： [`BigInt`](https://github.com/tc39/proposal-bigint) 是 JavaScript 中的一个新的数字基本类型，可以用任意精度来表示整数，使用 `BigInt` 可以安全地存储和操作大整数，即使这个数已经超过 `Number` 能表示的安全整数范围。
来看看上述问题使用 `BigInt` 处理的结果：

```js
// Chrome Version 68.0.3429.0 canary
const a = BigInt('9999910000199999');

console.log(a); // 9999910000199999n
console.log(Number(a % 1337n)) // 957 ✅
```

这次我们可以得到正确的答案了。在浏览器完全支持前，可以使用 Babel 7.0 来实现，它的内部是自动转换成 [big-integer](https://github.com/peterolson/BigInteger.js) 来计算，要注意的是这样能保持精度但运算效率会降低。
