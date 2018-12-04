---
title: webAssembly浅析
date: 2018-12-04
tags:
- Syntactic sugar
---

随着“大前端”概念的普及，前端能干的事情越来越多，涉猎各个领域，机器学习、WebVR、3D游戏、图像识别等等。众所周知，目前前端代码更多的运行环境还是在浏览器中，如何让诸如VR、3D游戏、复杂计算等这一类非常消耗性能的“骚东西“安稳地在浏览器中运行呢？有需求就有解决方案，前端大佬们从来不会让人失望的，下面让我们来重温下`webAssembly`的“诞生之路”。

<!-- more --->

## 一、JavaScript性能追求之路

进入主题前肯定要先了解下“历史”嘛。

### 1. JavaScript的性能问题

JS语言的简介与边解析边执行等基础知识在此就不多赘述了，直接上图:
![传统JS引擎](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_6bde3033-3458-4c4a-aeb5-9f5519542379.png)
![现代JS引擎](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_f2f45122-f3e5-4bad-b7ee-d805c20eead2.png)

对比上面两幅图可以感觉出前端开发者对于JS性能提升所做的努力。如果感觉不出请看下文:

```
// 传统引擎可以满足我们不同传参进行不同运算
function add (a) {
  return a + 1;
}
add(5);
add('50');

// 但是当多次重复进行某项“动作”时，传统的边解析边执行（重复解析）很浪费资源，这时候引入了JIT的现代引擎就能很好的解决这个问题
function add1(a, b) {
  return a + b;
}
for (let i = 0; i < 100000; i++) {
  add1(2, 3);
}

// 但是当出现下面这样的情况,现代引擎则会出现性能退化，也就是图2中的红虚线
for (let i = 0; i < 100000; i++) {
  i < 5000 ? add1(2, 3) : add1('哈哈', '呵呵');
}

// 还在想为什么？这就如同你辛辛苦苦结合上千次的开发经验封装出一套“无敌”模版，产品经理这个时候突然告诉你公司业务变了，以前的模式不适用了得重构。（自行体会）

```

### 2. JavaScript性能的探索

作为前端开发的主力核心，JS出现性能问题不可能没人管的。各大互联网巨头公司都有做过尝试:

- 微软的 TypeScript 通过为 JS 加入静态类型检查来改进 JS 松散的语法，提升代码健壮性；
- 谷歌的 Dart 则是为浏览器引入新的虚拟机去直接运行 Dart 程序以提升性能；
- 火狐的 asm.js 则是取 JS 的子集，JS 引擎针对 asm.js 做性能优化。

很可惜上述解决方案均未得到广大前端爱好者的“认可”。为了把话题引回到主题上，我这里简单介绍下火狐的`asm.js`。

### 3. asm.js与Emscripten编译器

`asm.js`：JS的严格子集，变量一律是静态类型且取消了垃圾回收机制；仅支持32位带符号整数与64带符号浮点数两种数据格式。
基本语法：
```
var a = 1
var x = a | 0 // 32位整数
var y = +a // 64位浮点数

function add3 (x, y) {
  x = x | 0
  y = y | 0
  return (x + y) | 0
}
```
很明显，虽然`asm.js`能手写但是我想没有几个前端程序员能忍受这种编程风格吧，所以`asm.js`不是我们直接的输出目标。
因此我们需要Emscripten编译器这样的一款能够生成`asm.js`的工具。一款能将`C/C++`编译成`asm.js`的编译器。主要的工作原理如下图所示：
![原理1](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_57f8913f-125d-40a8-8e92-8daa86427ed9.png)
![原理2](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_a2f6155b-959a-4dda-8e91-6a64e4115ca9.png)
安装SDK与使用的命令如下：
```
// 完整
$ git clone https://github.com/juj/emsdk.git
$ cd emsdk
$ ./emsdk install latest
$ ./emsdk activate latest

// just for fun
$ git clone https://github.com/juj/emsdk.git
$ cd emsdk
$ ./emsdk install --build=Release sdk-incoming-64bit binaryen-master-64bit
$ ./emsdk activate --build=Release sdk-incoming-64bit binaryen-master-64bit

// 安装完之后
$ source ./emsdk_env.sh --build=Release

// 创建一个c/c++文件后(如: hello.c)
$ emcc hello.c
$ node a.out.js
```

*注意：MAC下要确保已经安装了Git、CMake、Python、Xcode*

## 二、webAssembly的诞生

尽管`asm.js`大大提升了js的性能且具有一定的可读性，但是其解析编译时间，模块文件大小依旧不能满足追求完美的我们。于是，号称编辑更快、文件更小、执行更快的WASM诞生了。作为高级语言的一种编译目标，它拥有更好的移植性与适用性。

### 什么是webAssembly？

官方定义：
![wasm官方定义](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_cf872d39-4be8-431f-80a1-6005cccf97de.png)

通俗地来说它是一种新型的可移植、高性能、可读与可调试的字节码格式，而不是一种框架或是语言。目前也主要是通过Emscripten编译器将`C/C++`等高级语言编译成`.wasm`二进制模块。大致工作流程如下图：
![c—>wasm](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_918803a4-0c69-457a-8d7c-b3221a1863f0.png)

### WASM入门例子

由于同样是基于Emscripten编译器，那么我们就直接开始编写例子。
```
// test.c
#include <stdio.h>

int main(int argc, char ** argv) {
  printf("Hello World!\n");
  return 0;
}

// 命令行
$ emcc test.c  -s WASM=1 -o test1.html
```
这时候你会看到生成的相关文件。
![结果图1](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_56ff2e58-95bf-490a-b18a-4dcb4fd74db6.png)

没错就是这么简单！

### WASM初步探究

由于入门例子过于简单,在此补充下`.wasm`模块的编写与导入/导出。
WASM模块：
![WASM模块](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_123fa50e-92ae-4f4f-b2e1-ef284156b907.png)

说到`.wasm`模块的编写是件让人很头疼的事情，因为它是二进制文件，总不可能让我们这些习惯了使用高级语言的程序员一夜之间回到“远古时代”使用“0”，“1”书写‘hello world’吧。显然，不存在的。黑科技`WABT`能将`.wasm`与`.wat`相互转换。而后者则是我们读得懂的代码。大概长这样：
![wat](http://cdn.meishakeji.com/blog/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_0f2f765a-9018-4583-bd2d-915ff9be4c37.png)
与其说`.wasm`模块的编写其实是编写`.wat`文件，命令如下：
```
$ wasm2wat test1.wasm -o test1.wat // .wasm -> .wat
$ wat2wasm test.wat -o simple-test.wasm // .wat -> .wasm
```

简单模块封装例子如下：（console.log）
```
// simple-test.wat
(module
  (import "console" "log" (func $log (param i32)))
  (func (export "cLog") (param $p i32)
    get_local $p
    call $log
  )
)

// 命令行
$ wat2wasm simple-test.wat -o simple-test.wasm

// test.js
let importObject = {
  console: {
    log: function (num) {
      console.log(num)
    }
  }
}

fetch('./simple-test.wasm')
.then(res =>  res.arrayBuffer()) // 读取二进制文件
.then(bytes => WebAssembly.instantiate(bytes, importObject)) // 实例初始化WebAssembly
.then(modle => { // 调用实例上暴露的方法
  modle.instance.exports.cLog(10)
  // 打印10
})
```

## 三、总结

个人理解能力与研究深度有限，故文章中有任何不妥，欢迎指出！
