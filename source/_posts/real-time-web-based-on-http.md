---
title: 基于 HTTP 的实时 Web 通信
date: 2018-03-26 20:00:14
tags: 
- http
- js
---

## 前言

Web 应用的信息交互过程通常是客户端通过浏览器发出一个请求，服务器端接收和审核完请求后进行处理并返回结果给客户端，然后客户端浏览器将信息呈现出来，这种机制对于信息变化不是特别频繁的应用是能够满足的，但是对于那些实时要求比较高的应用来说，当客户端浏览器准备呈现这些信息的时候，这些信息在服务器端可能已经过时了。所以保持客户端和服务器端的信息同步是实时 Web 应用的关键要素，本文将简要介绍三种基于 HTTP 的实时 Web 通信方案。

<!-- more -->

## 短轮询 (Short Polling)

短轮询是最早的一种实现实时 Web 应用的方案。客户端以一定的时间间隔向服务端发出请求，以频繁请求的方式来保持客户端和服务器端的同步。这种同步方案的最大问题是，当客户端以固定频率向服务器发起请求的时候，服务器端的数据可能并没有更新，这样会带来很多无用的网络传输，所以这是一种非常低效的实时方案。

短轮询的优点：

- 客户端和服务器端实现都比较简单

短轮询的缺点：

- 请求间隔小，则频繁请求，无用请求多，浪费网络带宽和服务器资源
- 请求间隔大，则信息时效性一般

![短轮询](http://p652g7ewh.bkt.clouddn.com/%E7%9F%AD%E8%BD%AE%E8%AF%A2.png)

客户端短轮询示例：

```javascript
function poll() {
  axios.get('/api/shortPolling')
    .then(response => {
      // do something ...
      // 3秒后再发请求
      setTimeout(() => {
        poll();
      }, 3000);
    })
    .catch(error => {
      // error
    });
}
```



## 长轮询 (Long Polling)

长轮询是对短轮询的改进和提高，目地是为了降低无效的网络传输。当服务器端没有数据更新的时候，连接会保持一段时间直到数据或状态改变或者时间过期，通过这种机制来减少无效的客户端和服务器间的交互。当然，如果服务端的数据变更非常频繁的话，这种机制和短轮询比较起来没有本质上的性能的提高。

长轮询的优点：

- 有数据更新时，能比较及时的响应
- 没有新数据更新时，服务端会保持连接直到超时，请求相对少，节省网络带宽

长轮询的缺点：

- 服务器端需要保持 HTTP 连接，消耗一定的资源
- 服务器端数据变更频繁时，请求也会变得频繁

![长轮询](http://p652g7ewh.bkt.clouddn.com/%E9%95%BF%E8%BD%AE%E8%AF%A2.png)

客户端长轮询示例：

```javascript
function poll() {
  axios.get('/api/longPolling')
    .then(response => {
      // do something ...
      poll();
    })
    .catch(error => {
      // error
    });
}
```



## 服务器发送事件 (Server-Sent Events)

服务器发送事件（SSE）是客户端通过 HTTP 连接自动从服务器接收更新的技术。

服务器端要按照规定的格式返回响应内容，响应的内容类型为“text/event-stream”。响应文本的内容可以看成是一个事件流，由不同的事件所组成。每个事件由事件类型（event）和数据（data）两部分组成，同时每个事件可以有一个可选的标识符（id）。不同事件的内容之间通过一个空行来分隔，每个事件的数据可能由多行组成。客户端不再是使用浏览器提供的 `XMLHttpRequest` 对象来实现，而是使用 `EventSource` 对象实现，通过该对象建立连接和接收处理服务器推送的事件。

SSE 的优点：

- 实时性较好
- 除了超时的情况，客户端一般只要发一次请求

SSE 的缺点：

- 服务器端需要保持 HTTP 连接，消耗一定的资源
- IE 和 Edge 浏览器不支持

![服务器发送事件](http://p652g7ewh.bkt.clouddn.com/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%8F%91%E9%80%81%E4%BA%8B%E4%BB%B6.png)

客户端创建一个连接到服务器接收事件，示例如下：

```javascript
// 实例化 EventSource 对象，并指定一个 URL 地址
const eventSource = new EventSource('/api/serverSentEvents');
// 使用 addEventListener() 方法监听事件
eventSource.addEventListener('message', (e) => {
  // 消息数据
  const data = e.data;
  // do something ... 
}, false);
```

### 事件流格式

事件流仅仅是一个简单的文本数据流，每条消息后面都由一个空行作为分隔符，以冒号开头的行是注释行。服务器可以定期发送一条注释行消息，以保持连接防止连接超时。

```
: 这是注释行

data: 一条没有命名事件的消息数据

data: 一条多行
data: 消息数据

: 下面是事件名，浏览器在接收时，会产生对应事件名的事件
event: update
data: {"name": "harbor", "comment": "这是 JSON 格式的消息"}
```

除了上面看到的 `event` 和 `data` 字段，还有 `id` 和 `retry` 字段。

如果服务器设置了 `id` ，客户端会保持最近的事件 ID。如果连接中断了，重新连接发起新的请求时，会把最近接收到的事件 ID 放入 “Last-Event-ID” 请求的首部中，从而告知服务器最近的事件 ID。

```
: 下面是事件 ID
id: 8888
data: 一条消息数据
```

默认情况下，客户端会在连接断开后3秒尝试进行重新连接。这个时间也可以由服务器来指定，以毫秒为单位。

```
: 指定重新连接时间，单位毫秒
retry: 6000
```

### Node.js 服务器端的例子

服务器端使用 Node.js 实现 SSE 的简单示例：

```javascript
const http = require('http');

http.createServer((request, response) => {
  const filePath = request.url;
  if (filePath === '/api/serverSentEvents') {
    // Content-Type 首部指定为"text/event-stream"
    response.writeHead(200, {
      'Content-Type': "text/event-stream",
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive'
    });
    // 简单模拟服务器检查更新，每隔5秒发送一条消息
    setInterval(() => {
      const data = { timeStamp: Date.now() }; 
      // 返回一条简单的消息，JSON 格式
      response.write(`data: ${JSON.stringify(data)}\n\n`);
    }, 5000);
  } else {
    response.writeHead(404);
    response.end();
  }
}).listen(8080);
```

