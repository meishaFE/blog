---
title: Web服务器 - Nginx介绍与使用
date: 2018-08-01 14:00:00
tags:
- Web服务器
---

选择一个合适的Web服务器是每个建站站长在考虑网站性能必然要考虑的问题，目前市场上的Web服务器比较主流且具有代表性的非Apache和Nginx莫属。本文首先对Apache和Nginx进行简单的介绍和对比，然后再着重对Nginx服务器进行介绍和配置的解析。

## Apache 与 Nginx
就服务器的使用目的而言，Apache和Nginx都是用作 **接收用户的请求**  => **处理请求** => **将处理的结果返回给用户** 。两者的区别主要有以下两点：
<!-- more -->
#### 1.连接的处理
Apache和Nginx最大的不同在于它们对连接的处理方式。Apache提供一系列多重处理模块（mpm_prefork、mpm_worker、mpm_envent），通过这些多重处理模块对进程和线程池进行管理，以多进程或多线程 **同步** 、 **阻塞** 的方式来使用操作系统的资源，当访问人数过多时容易造成访问过慢。与Apache不同，Nginx的工作是多进程单线程的，通过 **异步** 、 **非阻塞** 的事件驱动的方式来实现的，使用多进程模式，不仅能提高并发率，而且进程之间相互独立，一个 worker 进程挂了不会影响到其他 worker 进程。
#### 2.内容的处理
这里的内容主要分为静态内容和动态内容，当客户端做出一个请求，如：
```http
GET /index.html
```
这时服务器返回的内容的形式（动态、静态）取决与服务器是否具有生成动态内容的能力。
如我们需要在返回的index.html资源中动态的显示当前时间：
```http
<!doctype html>
<html>
  <body>
    现在是 2018年08月03日 周五 10:40
  </body>
</html>
```
每次获取到的index.html的内容都是服务器动态生成的内容。而一般情况下，我们所获取到的index.html是固定存放在服务器上的静态内容。
```http
<!doctype html>
<html>
  <body>
    Hello World！
  </body>
</html>
```

**简单来说：**
**Apache拥有丰富的模块组件支持，动态内容处理强。**
**Nginx占用资源少，高并发处理强，静态内容处理高效。**

## Nginx反向代理与配置

#### Nginx反向代理
当客户端向服务端的反向代理发起请求时，反向代理以某种负载均衡机制将请求分发到各个目标服务器，并且将这些服务器所处理返回的内容返回给客户端。这个反向代理服务器没有保存任何网页的真实数据，所有的静态网页或者CGI程序，都保存在内部的Web服务器上。因此对反向代理服务器的攻击并不会使得网页信息遭到破坏，这样就增强了Web服务器的安全性，其中CDN就是一个比较常见的例子。

#### Nginx的配置
Nginx的配置文件一共由三个部分组成，**全局块**、**events块** 和 **http块**。其中，**http块**中由包含了 **http全局块**、**多个server块**。


**全局块**是默认配置文件从开始到events块之间的一部分内容，主要设置一些影响Nginx服务器整体运行的配置指令，因此，这些指令的作用域是Nginx服务器全局。
通常包含配置运行Nginx服务器的用户（组）、允许生成的worker process数，Nginx进程PID存放路径、日志的存放路径和类型以及配置文件引入等。
```conf
# 全局块
user root; #运行Nginx服务器的用户（组）
worker_processes  1; #允许生成的worker process数

error_log  logs/error.log; #日志的存放路径和类型
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#pid        logs/nginx.pid; #Nginx进程PID存放路径
#include conf.d/* #配置文件引入
error_log  logs/error.log; #日志的存放路径和类型
```

**events块**主要影响Nginx服务器与用户的网络连接。常用到的设置包括是否开启对多worker process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型处理连接请求，每个worker process可以同时支持的最大连接数等。
这一部分的指令对Nginx服务器的性能影响较大，在实际配置中应该根据实际情况灵活调整。
```conf
# events块
events {
  worker_connections  1024; #单个worker进程最大连接数
  use epoll;#epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型
}

```

**http块**是Nginx服务器配置中的重要部分，代理、缓存和日志定义等绝大多数的功能和第三方模块的配置都可以放在这个模块中。
http块中可以包含自己的全局块，也可以包含server块，server块中又进一步包含location块。
可以在http全局块中配置的指令包括文件引入、MIME-Type定义、日志自定义、是否使用sendfile传输文件、连接超时时间、单请求数上线等。

```conf
# http块
http {
  include       mime.types; #文件扩展名与文件类型映射表
  default_type  application/octet-stream; #默认文件类型

  #设定访问日志的日志记录格式
  #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
  #                  '$status $body_bytes_sent "$http_referer" '
  #                  '"$http_user_agent" "$http_x_forwarded_for"';

  #access_log  logs/access.log  main; #访问日志的存放路径和类型

  sendfile        on; #开启高效文件传输模式，实现内核零拷贝
  #tcp_nopush     on; #激活tcp_nopush参数可以允许把httpresponse header和文件的开始放在一个文件里发布，积极的作用是减少网络报文段的数量

  #keepalive_timeout  0; #连接超时时间，单位是秒

  #gzip  on; #开启gzip压缩功能
}
```

**负载均衡配置**

```conf
server {
  listen       80;#监听的端口，也可以是172.16.1.7:80形式
  server_name  gin-educationadmin.meishakeji.com;#代理的服务域名
  location / {
    proxy_pass http://gin-educationadmin.meishakeji.com; #将访问gin-educationadmin.meishakeji.com的所有请求都发送到upstream定义的服务器节点池。
    proxy_set_headerHost  $host; #在代理向后端服务器发送的http请求头中加入host字段信息，用于当后端服务器配置有多个虚拟主机时，可以识别代理的是哪个虚拟主机。这是节点服务器多虚拟主机时的关键配置。
    proxy_set_header X-Forwarded-For$remote_addr; #在代理向后端服务器发送的http请求头中加入X-Forwarded-For字段信息，用于后端服务器程序、日志等接收记录真实用户的IP，而不是代理服务器的IP。
    proxy_connect_timeout60; #设定反向代理与后端节点服务器连接的超时时间，即发起握手等候响应的超时时间。
    proxy_send_timeout 60; #设定代理后端服务器的数据回传时间
    proxy_read_timeout 60; #设定Nginx从代理的后端服务器获取信息的时间
    proxy_buffer_size 4k; #设定缓冲区的大小
    proxy_buffers 4 32k; #设定缓冲区的数量和大小。nginx从代理的后端服务器获取的响应信息，会放置到缓冲区。
    proxy_busy_buffers_size 64k; #设定系统很忙时可以使用的proxy_buffers大小
    proxy_temp_file_write_size 64k; #设定proxy缓存临时文件的大小
  }
  access_log off;#反向代理如果并发大，务必要关闭日志，否则IO吃紧。
}
```