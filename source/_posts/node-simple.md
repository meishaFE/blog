---
title: Node.js浅谈之FS API
date: 2018-08-27
tags:
- JavaScript
- Node.js
---

晃眼Node走过了九个春秋了,在这九年内一直发挥着它强有力的作用,npm生态圈持续影响着我们“切图仔”的工作方式和工作效率。尽管今年Node之父Ryan Dahl发布了新的项目“Deno”,但是Ry此次时隔九年的“壮举”并非外界传得那么恐怖。“Deno”目前为止仅仅只是一个demo,并不是下一代Node.js。所以不要惊慌,好好学习Node！

## 一、阻塞与非阻塞IO
在讲FS(文件系统)之前,先简单谈谈Node的重要特性,也方便理解后续的API。

### 1. 阻塞

学过Java和PHP的同学应该都知道，当我们调用sleep函数让程序进程“睡眠”时,该函数之后的语句在这段时间内是不会被执行的,它们将会等待这段“睡眠“时间过去之后再继续执行,这就是阻塞。

```
// PHP
print('hello');
sleep(5);
print('world');

// 执行结果
hello
(等待5s后)
world
```

### 2. 非阻塞

然而Node不同的地方在于它采用事件轮询的机制实现了非阻塞。它会先“跳过”这个会阻塞程序代码执行的语句去执行后面的代码。本质上它会先注册每个事件,然后不停地询问内核这些事件是否分发, 每分发一个事件就会触发对应的回调函数。如果没有事件触发就按正常的执行顺序执行其他代码,直到新的事件触发,再去执行对应的回调函数。由此可见,异步非阻塞是Node重要的特性之一,而回调是常用的书写形式。

```
// node
console.log('hello');
setTimeout(function () {
  console.log('world');
}, 5000);
console.log('hi');

// 执行结果
hello
hi
(等待5s后)
world
```

## 二、FS API及实践

前面介绍了Node的回调与事件机制,这小节将通过一次基于非阻塞事件的I/O编程实践来介绍处理进程的stdin和stdout以及文件系统(fs)相关的API。

### uav-check实现过程

依稀记得那是去年的寒冬,经历了秋招“0 offer”惨痛失败的我意外地发现了一家公司的“冬招”(春招提前批)。然后给我布置了一道编程作业,要求如下图：
![题目](http://meishakeji-oss1.oss-cn-shenzhen.aliyuncs.com/Pr/uav-check/20180319235302753)

很明显地,作为一名前端爱好者,既然能使用JavaScript编程实现那么我是绝对不会使用其他语言的。正好那段时间迷上了Node,这是一次绝佳的实践机会。废话不多说,进入正题。

首先,作业的要求就是实现一个能录入无人机坐标数据并通过用户输入的ID返回对应查询结果的简单交互程序。为了尽可能的降低开发成本不小题大作,所以我选择Node来构建一个简单的命令行工具(CLI)。程序基本结构如下图所示：

![结构](http://meishakeji-oss1.oss-cn-shenzhen.aliyuncs.com/Pr/uav-check/1535296425979.jpg)

主要分为三个部分：控制主流程的index.js、负责读取本地磁盘上数据文件并处理得到一个JS对象的importUavMsg.js和根据用户输入返回对应结果的queryUavMsg.js。下面依次展开讲解。

#### index.js

如何根据用户的输入响应不同流程操作,引导用户进行操作是这个js文件要做到的功能。根据题意可知用户进入程序第一步则需要指定所要录入的数据文件的路径,然后再输入所要查询的ID。具体流程图如下：

![流程图](http://meishakeji-oss1.oss-cn-shenzhen.aliyuncs.com/Pr/uav-check/%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

由于涉及到用户主动输入的操作,那么就需要使用到上述所说的`process.stdin`和`process.stdout`这两个api。因为是挂载在全局变量`process`下,所以无需`require`其他模块。

```
// 模块声明

process.stdin.setEncoding('utf8') // 设置字符集

const stdin = process.stdin
const stdout = process.stdout
const platform = process.platform // 运行时操作系统的信息

```

和大多数编程语言一样输入输出流用起来都是很简单的,在Node中使用`process.stdout.write('xxxx')`来输出结果到终端,使用`process.stdin.on('readable', [callback])`来接受输入流,当然这里的'readable'可以替换成'data'等其他事件。由于Node的特点就是异步回调,所以这次实践相关API的使用我都采用其异步方法然后通过回调函数的形式来实现相关功能,同学们在日常开发过程中可根据官方文档的描述结合实际应用场景采用合适的方式。这里我是这么实践的

```
const importMsg = require('./importUavMsg').func
const queryMsg = require('./queryUavMsg')

let arr // 无人机的数据对象
let stdinType = 'path' // 输入的字符串的类型，'path'代表输入的是数据文件路径，'num'代表查询的信息序号
let isExiting = false // 标识用户是否进入退出程序阶段
stdout.write('请输入数据文件的相对路径：\n')
stdin.on('readable', () => {
  let chunk = stdin.read()
  if (typeof chunk === 'string') {
    // 判断操作系统
    if (platform === 'darwin') {
      chunk = chunk.slice(0, -1) // Mac下回车占一个字符
    } else {
      chunk = chunk.slice(0, -2) // Windows下回车占两个字符
    }
  }
  // 退出流程
  if (chunk === '' || chunk === 'exit') {
    stdout.write('您是否要退出Y/N：\n')
    isExiting = true
  }
  if (isExiting && chunk !== '' && chunk !== 'exit') {
    if (chunk.match(/^y(es)?$/i)) {
      stdin.emit('end')
    } else {
      stdinType === 'path' ? stdout.write('请输入数据文件的相对路径：\n') : stdout.write('请输入消息的ID：\n')
      isExiting = false
    }
    return 0
  }
  if (chunk !== null && chunk !== '' && chunk !== 'exit') {
    if (isNaN(chunk - 0)) {
      if (stdinType === 'num') {
        stdout.write('参数错误,请输入消息的ID(数字)：\n')
      } else {
        importMsg(chunk, data => { // 读取数据文件
          if (data.isError) {
            stdinType = 'path'
            stdout.write('请输入正确的路径：\n')
          } else {
            stdinType = 'num'
            arr = data
            process.stdout.write('请输入消息的ID：\n')
          }
        })
      }
    } else {
      if (stdinType === 'path') {
        stdout.write('参数错误,请输入数据文件的相对路径：\n')
      } else {
        queryMsg(arr.msg, chunk - 0) // 查询消息
      }
    }
  }
})
stdin.on('end', () => {
  stdout.write('已退出系统\n')
})
```

主要是通过输入流`chunk`的内容来进行不同操作的响应,具体流程在上面已经陈述。在这里卖个关子先,为什么导入无人机坐标信息的函数我封装成了回调的形式？

#### importUavMsg.js

介绍完主流程后,接下来简单介绍下我是如何读取磁盘上的数据文件的。Node之所以如此受欢迎除了它强大的非阻塞I/O能力之外,还得益于它对于一些常用模块的高度封装如Http网络模块和文件模块等。极大程度上提高了开发效率,下面我们来看看在Node中如何读取文件流。

```
const fs = require('fs')
fs.readFile('<directory>', 'utf8' ,(err, data) => {
  if (err) throw err;
  console.log(data);
});
```
你没看错,就是这么简单。不必再像Java那样new一个`FileInputStream`这么长类名的文件类去接受文件以及使用`try、catch`这样的错误捕获语句去解决错误问题,直接使用引入'fs'模块调用`readFile`方法读取文件,回调函数增加一个错误参数的方式解决错误。是不是觉得书写起来更加飘逸？反正我觉得是。

当然你也可以使用`readFileSync`同步地读取文件,只不过在捕获错误时你需要使用`try、catch`这样的语句去处理以及如果多人同时请求读取同一文件,很有可能会冲突挂起。

至于这次实践所采用的方式肯定没这么简单,因为我们需要一行一行地去读取数据,处理数据。所以可能会有点差异。也将引入新的模块——`readline`。官方的例子如下：

```
const readline = require('readline');
const fs = require('fs');

// 创建一个实例
const rl = readline.createInterface({
  input: fs.createReadStream('sample.txt'), // 创建输入流
  crlfDelay: Infinity
});

rl.on('line', (line) => {
  console.log(`文件的单行内容：${line}`);
});
```

同样地先创建一个实例,然后通过监听`line`事件来获取所读到的每一行数据流。实践代码如下：

```
let arr = {
  isError: false,
  msg: []
}
const importMsg = function (fileDir, callback) { // 利用回调解决异步返回值的问题
  let filePath = path.resolve(__dirname, fileDir)
  // 提前判断路径正确性
  fs.access(filePath, err => {
    if (err) {
      console.error('该文件不存在!\n')
      arr.isError = true
      callback(arr)
    } else {
      const rl = readline.createInterface({
        input: fs.createReadStream(filePath),
        crlfDelay: Infinity
      })
        ...
      rl.on('line', (line) => {
        ...
      }).on('close', () => {
        ...
        callback(arr)
      })
    }
  })
}
```

细心的同学肯定会发现实践的方式又和官方的代码有所出入,我又引入了一个新的方法`access`。没错,我使用该方法来检测传入的路径下文件是否存在从而解决文件不存在的问题。那么久有同学问了,之前不是说过可以通过回调函数`(err, data) => {if (err) throw err; ...}`的形式解决吗？确实Node所封装的绝大部分异步方法的回调函数第一个参数均是错误对象,我们都可以通过这种方式去做错误处理,而且也是被提倡的错误处理方法。但是很遗憾,我并没有发现`readline.createInterface`有这种机制。所以,大家在平时开发中使用新的方法时一定要先参阅相关文档说明,避免错误地'举一反三'。当然我也有尝试过去捕获根源`fs.createReadStream`的错误(确认过了它有这种机制),因为说到底错误是来源于它。但是尝试过几种写法后,依旧没有写出我满意的代码。(如果你们有谁有好的想法,可以私聊我。)

另一个注意的地方就是我上面买的关子,很明显`importMsg`这个函数传入了一个回调函数。正因为我所采用的是异步读取文件的方法,我返回到`index.js`中的数据对象一定是要在这边异步操作结束后才能返回的,不然就是空的,不信你们可以尝试下。(返回的`arr` 绝对是你声明时候的值)

#### queryUavMsg.js

这个就是简单的查询操作啦。就不多赘述了,直接看代码吧

```
const stdout = process.stdout
const queryMsg = function (msgArr, id) {
  if (id >= msgArr.length || id < 0) {
    stdout.write(`查询结果：Cannot find ${id}\n`)
    stdout.write('请输入消息的ID：\n')
  } else {
    if (msgArr[id].isError) {
      stdout.write(`查询结果：Error ${id}\n`)
      stdout.write('请输入消息的ID：\n')
    } else {
      let object = msgArr[id]
      stdout.write(`查询结果：${object.id} ${id} ${object.newPos[0]} ${object.newPos[1]} ${object.newPos[2]}\n`)
      stdout.write('请输入消息的ID：\n')
    }
  }
}
module.exports = queryMsg
```

### uav-check结果展示

话不多说,看图

![结果](http://meishakeji-oss1.oss-cn-shenzhen.aliyuncs.com/Pr/uav-check/%E7%BB%93%E6%9E%9C.png)

## 三、总结

距离上次学习Node还是上半年的事情了,知识过久了不去用真的会被遗忘掉。所以这次文章分享也是对自己的一个逼迫,重新拾起学过的知识,激发自己学习的动力。后续还会更新更多关于Node的进阶知识,敬请期待。

由于"时代久远",文章有啥错误的地方还请指出纠正！拜谢！