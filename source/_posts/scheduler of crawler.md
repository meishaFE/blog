---
title: 爬虫 - 中央调度器的实现分析
date: 2019-05-18 14:30:00
tags:
- node.js
- redis
---

## 前言
最近在学习一个基于nodejs的一个爬虫项目，`NEOCrawer`。
git地址：https://gitee.com/dreamidea/neocrawler

<img src="http://git.oschina.net/uploads/images/2014/0912/203424_dbbb3d02_13016.png"/>

<!-- more -->

上图是整个爬虫项目的架构图。

其中，有一个模块在最左边，叫做`Super Scheduler`，翻译过来的基本意思是：`中央调度器`。之前对调度器这个概念比较陌生，调度器在项目中担当了一个什么样的角色呢。于是在网上搜索了一下，摘抄了一段话：
`“进程是操作系统虚拟出来的概念，用来组织计算机中的任务。但随着进程被赋予越来越多的任务，进程好像有了真实的生命，它从诞生就随着CPU时间执行，直到最终消失。不过，进程的生命都得到了操作系统内核的关照。就好像疲于照顾几个孩子的母亲内核必须做出决定，如何在进程间分配有限的计算资源，最终让用户获得最佳的使用体验。内核中安排进程执行的模块称为调度器（scheduler）”。`
-- 来自 https://www.cnblogs.com/vamei/p/9364382.html

而在这个爬虫系统中，中央调度器是负责网址的调度（任务的派发由网站的权重来决定），爬虫进程通过调度器派发任务，实现分布式运行，单点故障不影响整体系统。

简单来说，一个分布式运行的爬虫系统，会从调度器调度好的待抓取队列中取出任务进行抓取，将发现的网址放入网址库，将摘取的内容存库。


<!-- more -->

## 调度配置

本项目中的中央调度器是结合了redis来实现的， neocrawler用到4个存储空间, `driller_info_redis_db`、`url_info_redis_db`、`url_report_redis_db`、`proxy_info_redis_db`。
而本文只探讨有关存储抓取规则及网址的`driller_info_redis_db`和存储网址信息的`url_info_redis_db`。

## driller_info_redis_db 
将项目运行起来后，我们可以在localhost:8888这个可视化配置界面中配置爬虫的抓取规则

对于`driller_info_redis_db`，一般的配置如下：
```js
{
  "domain": "sovxin.com",
  "url_pattern": "^http://www.sovxin.com/t_.*?.html$",
  "alias": "list",
  "extract_rule": {
    "category": "crawled",
    "rule": {}
  },
  "drill_rules": [
    "a"
  ],
  "priority": 3, // 调度优先级，数字越小越优先
  "weight": 10, // 调度权重，数字越大越有限
  "schedule_interval": 86400, // 重新调度的周期，单位秒
  "seed": [ // 种子地址，重新调度时从这些网址开始
    "http://www.sovxin.com/t_xiuxianyule_#.html#1#300#1"
  ],
  "schedule_rule": "FIFO" // 调度方式，FIFO或者LIFO
  ... // 配置的删减版
}
```

`这样的list类型，存储了某种匹配规则的网址队列, 爬虫发现符合抓取规则的网址时，将其存入相应的队列，调度器将从这些队列里摘取网址进行调度，爬虫依据调度队列进行抓取，整个过程循环反复。`


提交规则后，会将刚才的这一个配置以`key-value`的形式，存入`redis`，方便后面快速提取规则并进行匹配。
这样，在`redis`中，就会有一个`key`，为`driller:sovxin.com:list`；对应的`value`为刚才所配置的这些内容。



然后执行`node run.js -i "实例名称" -a schedule`，就能够抓取到符合刚才所定下的规则的网址队列，并存入`redis`中的`url_info_redis_db`。

调度器的执行首先会去获取对比配置里的权重优先级的值，并进行优先级排序，具体实现代码如下：

```js
scheduler.prototype.start = function(){
    var scheduler = this;
    // 这里监听了一个`priorities_loaded`的事件，当权重值加载完成了以后，就会对优先级列表进行排序
    this.on('priorities_loaded',function(priotity_list){
        this.priotity_list.sort(function(a,b){
            return b['rate'] - a['rate'];
        });
        this.doSchedule();
    });
    // trigger
    ...
}
```

获取到优先级列表之后，便会执行调度函数。在调度函数里，首先会判断调度队列的长度，如果当前执行到的索引大于调度队列的长度，则跳出调度，并在控制台中打印`schedule round finish`

以下是执行调度的核心代码：

```js
scheduler.prototype.doSchedule = function(){
    var scheduler =  this;
        scheduler.schedule_version = (new Date()).getTime(); // 记录抓取规则配置的版本信息
    var redis_cli = scheduler.redis_cli0; // 获取redis的调度队列实例
        redis_cli.llen('queue:scheduled:all',function(err, queue_length) {
            ...
            var index = -1;
            var left = 0;
            // whilst的作用是，用第一个参数（函数）来判断是否满足执行条件(这里是判断调度队列的长度是否满足大于目前执行到的队列索引值)，如果满足，则执行第二个参数（函数）；如果不满足，则执行第三个参数（函数）。
            async.whilst(
                function() { index++;return index < scheduler.priotity_list.length },
                function(cb) {
                    ...
                },
                function(err) {
                    if(err)logger.error(err);
                    logger.info('schedule round finish')
                }
            );

        });
}
```

爬虫/调度器对爬虫规则的变更是热感应(实时刷新)的, 一个周期大概间隔几秒，都将所有规则扫描重新载入, 于是就采用版本记录的方式, web配置中心更改抓取规则后变更版本信息, 爬虫会重复检测`schedule_version`这个键, 如果发现版本变化则重新加载抓取规则.

## url_info_redis_db

该空间存储了网址信息，key值如：9108d6a10bd476158144186138fe0ba8，hash类型, 记录了包括在哪个页面被发现、当前状态、爬虫系统对该网址的操作轨迹(发现-调度-抓取-存储/失败等等)以及最后操作时间. 这些记录是调度器对二次发现网址是是否调度抓取的依据.

核心代码如下：

```js

// 检查当前抓取到的url是否存入url_info_redis_db
scheduler.prototype.checkURL = function(url,interval,callback){
    var scheduler = this;
    var redis_cli0 = this.redis_cli0;
    var redis_cli1 = this.redis_cli1;
    ...
    redis_cli1.hgetall(kk,function(err,values){
        if(err){return callback(false);}
        ...
        scheduler.updateLinkState(url,'schedule', false, fn (bol) {
            // 在这里做判断，如：抓取到的链接是否需要存入redis
            // 如果抓取成功
            if(bol){
                redis_cli0.rpush('queue:scheduled:all',url,function(err,value){
                    if(err){
                        logger.warn('Append '+url+' to queue failure');
                        return callback(false);
                    }else{
                        logger.info('Append '+url+' to queue successful');
                        return callback(true);
                    }
                });
            }else return callback(false);
        });
    });
}
```

总体来说，在实现爬虫调度器中，主要通过入口网址对链接的抓取，然后判断链接是否符合规则，最终得到一个调度队列。通过设置定时更新，不断刷新调度队列，保证最新执行的任务是优先级最高的。

本文对作者实现的调度器进行了主要部分的解析，发现在调度器的实现过程中也是频繁的使用到回调函数，阅读起来比较困难，欢迎共同探讨。

最后，本文如有错漏，欢迎指正。