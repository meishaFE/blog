---
title: NodeJS之mockdata
date: 2018-10-2 23:49:55
tags:
- Node.js
- Http
---

## 用nodejs搭建一个虚拟服务器，模仿后台接口数据返回，实现mockdata

不知道各位前端小伙伴在开发过程中，有没有遇到过有过以下场景：
一个项目需求下来以后，前端的静态页面堆完了，后台的接口还没有给到。不知道各位平时在开发中是怎么处理这个问题的呢？
下面我就介绍一种方法：
1. 在需求定义设计清楚了以后，前后端统一制作出来一套接口文档（其中的接口名称，传入返回的参数名称以及数据类型都定义好）；
2. 前端开发完静态页面后，用nodejs搭建本地服务器，模拟接口路径，判断逻辑以及接口返回；
3. 待后端给到接口以后，再将真实接口替换；
这样就实现了脱离后台数据依赖，前端独立开发业务功能

<!-- more -->

### （一）准备环境

####  1. 确保已在本地安装 node 和 npm 
安装node 会自动安装npm包管理工具；
检查是否有安装node和 npm包管理工具，打开cmd命令行工具，输入node -v 和npm -v，会输出对应的版本号，如下所示：

```javascript
$ node -v
v8.4.0

$ npm -v
5.6.0
```
####  2. 安装express应用程序生成器

全局安装express-generator;

```javascript
npm install -g express-generator
```

### （二）项目初始化

用命令行工具进入当前文件夹，创建一个名为 mockInterData 的express应用程序

```javascript
express --view=pug mockInterData
```

成功后会自动在目标位置创建一个名为 mockInterData 的项目并生成很多文件，进入到这个工程当中会发现有个package.json文件，保存所有的依赖配置信息，在改项目下npm i 或者 cnpm i 来安装项目依赖。安装完成后，程序文件夹结构如下：

```javascript
.
├── app.js
├── bin
│   └── www
├── node_modules //安装的依赖
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.pug
    ├── index.pug
    └── layout.pug
```
再试下: $ npm start 

```javascript
$ npm start

> myapp@0.0.0 start E:\my_work\myapp
> node ./bin/www
```
在浏览器输入localhost:3000，会打开一个页面显示Welcome to Express，说明已经成功启动服务了。

### （三）自定义配置

细心的你可能会发现，浏览器打开localhost:3000/users,可以看到页面显示respond with a resource。然后找一下代码发现了这一段

```javascript
// routes/users.js文件

var express = require('express');
var router = express.Router();

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});

module.exports = router;
```

服务器收到“/”的请求，会返回一个字符串，当然你也可以自定义返回，例如json数据，是不是仿佛明白了什么？
。。。
没错，这样已经可以完成一个初步的自定义数据返回了。但是项目中，每个接口都写个逻辑返回，未免也太麻烦。这里我就借鉴了一下别人的方法：
在项目下新建一个config文件夹并新建一个api.js,配置一下：

```javascript
// config/api.js

var fs = require('fs');

/**
 * 检查请求的路径是否存在
 * @param apiName 请求路径
 * @param method  请求方式
 * @param params   请求参数
 * @param res 返回请求
 */
function getDataFromPath (apiName,method,params,res){
    if(apiName){
        fs.access(
            // 提取请求路径中的js文件
            apiName.substring(1)+'.js',
            // 回调函数，检查请求的路径是否有效失败返回一个错误参数
            function(err){
                if(!err){
                    // 每次请求都清除模块缓存重新请求
                    delete require.cache[require.resolve('..'+apiName)];
                    try{
                        addApiResult(res,require('..'+apiName).getData(method,params));
                    }catch(e){
                        console.error(e.stack);
                        res.status(500).send(apiName+' has an error,please check the code.');
                    }
                }else{
                    addApiResult(res);
                }
            }
        );
    }else{
        addApiResult(res);
    }
};
/**
 *  响应头
 * @param res
 */
function addApiHead(res){
    res.setHeader('Content-Type', 'application/json;charset=utf-8');
    // 跨域
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Headers', 'Content-Type, Content-Length, Authorization, Accept, X-Requested-With , yourHeaderFeild');
    res.header('Access-Control-Allow-Methods', 'PUT, POST, GET, DELETE, OPTIONS');
    // 控制http缓存
    res.header("Cache-Control", "no-cache, no-store, must-revalidate");
    res.header("Pragma", "no-cache");
    res.header("Expires", 0);
}

/**
 * 返回参数，如无返回参数返回404
 * @param res
 * @param result
 */
function addApiResult(res,result){
    if(result){
        res.send(result);
    }else{
        res.status(404).send();
    }
}

/*请求方式*/
// get
exports.get = function(req, res){
    addApiHead(res);
    getDataFromPath(req.path,'GET',req.query,res);
};

// post
exports.post = function(req, res){
    addApiHead(res);
    getDataFromPath(req.path,'POST',req.body,res);
};
```

在app.js中引入api.js， 再做一些小的修改

```javascript
//引入API
var api = require('./config/api');

/*配置请求修改*/
app.get('/', function(req, res){
    res.send('hello world');
});
app.get('/api/*', api.get); // 调用api的get post方法
app.post('/api/*', api.post);
```

然后你就可以根据你的项目接口名称来配置自己的“小型服务器”了
例如：你的接口名称是："/project/list/enums", 那你就可以在当前项目的根目录中新建一个api文件夹/list文件夹/enums.js

```javascript
│ api
│ └── list
│     └── enums.js
```

开发的时候指向本地服务器器接口，联调测试上线的时候只需要把指向本地服务器地址替换成线上地址一下就可以了。

```javascript
if (dev) {
  BASE_URL = 'http://localhost:3000/api/';
} else {
  BASE_URL = 'http://www.***.com/';
}
```


可能有的小伙伴儿会提到跨域问题，这个就需要nginx来帮忙了。
