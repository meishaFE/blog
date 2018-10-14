---
title: HTTP Basic 认证
date: 2018-06-10 23:15:27
tags:
- http
---

Basic 认证是 Web 服务器于客户端之间进行认证的一种方式， 最初是在[HTTP 1.0](https://tools.ietf.org/html/rfc1945#section-11) 规范（[RFC 1945](https://tools.ietf.org/html/rfc1945)）中定义，后续的有关安全的信息可以在HTTP 1.1规范（[RFC 2616](https://tools.ietf.org/html/rfc2616)）和HTTP认证规范（[RFC 2617](https://tools.ietf.org/html/rfc2617)）中找到。

## Basic 认证过程

当客户端请求了需要进行 Basic 认证的资源，服务器就会返回带有 401 unauthorized 状态码以及 `WWW-Authenticate` 首部字段的响应。`WWW-Authenticate` 内容包含了认证方式（Basic）及 Request-URI 安全域字符串（realm）。

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic realm="My realm"
```
<!-- more -->
客户端收到 401 状态码并要求进行 Basic 认证后，就需要用户 ID 和密码发送给服务器，发送的字符串内容是由用户 ID 和密码通过冒号（:）连接后，再经过 Base64 进行编码处理得到。然后把这个字符串写入 `Authorization` 首部字段，并发送请求。

客户端是浏览器的情况下，这个阶段会弹出对话框，要求输入用户 ID 和密码，确定后就会自动完成编码发送请求。

```http
GET /index.html HTTP/1.1
Host: localhost
Authorization: Basic aGFyYm9yOmhhcmJvcg==
```

服务器接收到包含 `Authorization` 字段的请求后，会对其正确性进行验证。如验证通过，则返回一条包含 Request-URI 资源的响应。

## Basic 认证优点

在假定客户端和服务器之间的连接安全的情况下，通过Basic 认证来实现身份认证非常简单，只需要对Web服务器（比如Apache、NGINX）简单的配置即可轻松实现。客户端如果是浏览器，也无需做什么处理。

## Basic 认证缺点

Basic 采用了 Base64 编码处理，但这并非加密处理，可以轻易地进行解码。在没有进行加密的 HTTP 通信中使用 Basic 认证，可以轻易被窃听而得到认证的用户 ID 和密码。

在Basic 认证过后，只能通过关闭浏览器进行认证注销操作，这样切换用户时就会比较麻烦。