---
title: PM2 入门
date: 2019-07-06 15:00:00
tags: node
---

## PM2是什么
PM2是一个进程守护的管理器，可以帮助我们管理并且保持应用程序的在线状态

<!-- more -->

## 安装PM2
我们可以通过NPM或者Yarn来全局安装最新的PM2版本，安装命令如下：
```bash
npm install pm2@latest -g
or
yarn global add pm2
```

## 使用PM2
安装完pm2之后，我们就可以在我们的项目中去使用了；最简单的启动方式是通过以下命令：
```bash
pm2 start app.js
```
当然，其他的应用程序也可以通过此命令来进行启动；同时，我们还可以给CLI加入一些常用的选项指令
```bash
 # 指定当前应用名称
--name <app_name>

# 监听文件变化
--watch

# 设置程序重新加载的阈值
--max-memory-restart <200MB>

# 指定日志文件
--log <log_path>

# 给脚本传递额外的参数
-- arg1 arg2 arg3

# 自动重启的延迟时长
--restart-delay <delay in ms>

# 给日志添加时间前缀
--time

# 不自动重启程序
--no-autorestart

# 排除监听的目录/文件
--ignore-watch

# 启用的实例个数，可用于负载均衡
-i  --instances
```
以上参数我们可以配合着我们的命令一起使用，常用的一些命令有：
```bash
# 启动
pm2 start app_name

# 查看详细状态信息
pm2 show app_name

# 重启
pm2 restart app_name

# 停止
pm2 stop app_name

# 删除
pm2 delete app_name

# 停止所有
pm2 stop all

# 查看所有进程
pm2 list

# 查看所有的进程状态
pm2 status

# 查看某一个进程的信息
pm2 describe app_name

# 显示日志
pm2 logs

# 查看基于终端的实时仪表板
pm2 monit
```
启动我们的一个官网后查看详细信息如下：
![](https://lexiangla.com/assets/f306a01a9e7611e9be3c0a58ac131118)

## 生态系统文件
通常，我们在管理一个程序时，需要一系列的配置来微调我们每个应用程序的行为，此时，我们可以通过生态系统文件来管理。
要生成一个示例流程文件，可以通过以下命令：
```bash
pm2 ecosystem
```
通过此命令，我们将得到一个样本`ecosystem.config.js`  配置支持的格式包括`Javascript`,`JSON`,`YAML`
```javascript
module.exports = [
	apps: [
		name: "app",
		script: "./app.js",
		env: {
			NODE_ENV: "development",
		},
		env_production: {
			NODE_ENV: "production"
		}
	]
]
```
然后，我们可以通过以上介绍的命令和参数来修改这个样本来达到我们自己的需求，并且可以设定不同的环境指令；要注意的是,使用Javascript配置文件时需要使用的结束文件名为`.config.js`。
### YAML 格式
```yaml
 apps:
 	- script        :  ./api.js
	  name         : 'api-app'
	  instances   : 4
	  exec_mode: cluster
	 - script   : ./worker.js
	 	name   : 'worker'
		watch  : true
		env     :
			NODE_ENV : development
		env_production: 
			NODE_ENV: production
```

然后，我们就可以轻松的运行并且管理我们的流程，当然，我们还可以使用其名称和选项对特定的应用程序执行操作` --only <app-name>`:

```bash
pm2 start ecosystem.config.js --only my-app
pm2 restart ecosystem.config.js --only my-app
pm2 reload ecosystem.config.js --only my-app
pm2 delete ecosystem.config.js --only my-app
```
## 负载均衡
```bash
# 根据机器CPU核数，开启对应数目的进程来运行项目
pm2 start app.js -i max
```
通过此条简单的命令运行起来的程序就可以轻松的实现负载均衡这一效果，可以说是很6了

## 监控可视化
通过安装`pm2-web`，我们就可以可视化的查看和分析监控数据
```bash
# 安装pm2-web
npm install -g pm2-web

# 使用
pm2-web --config pm2-web-config.json
```

## 重启策略
当我们应用发生异常时（数据库关闭等等），为了不是减少数据库和服务器的压力，而不是疯了一样的来重启我们的程序，我们需要制定一些重启的策略
```bash
pm2 start app.js --exp-backoff-restart-delay=100
```
通过此命令启动应用，并且应用程序意外崩溃时，会激活`--exp-backoff-restart-delay`选项，我们可以看到等待重启的新应用程序状态，此外，通过`pm2 logs`还可以看到重启的延迟时长会增加，其规则是：重启之间的重启延迟时长将以指数移动平均值增加的形式，直到重启之间的时长达到最大值15000ms.并且，当应用程序恢复到稳定模式时（正常运行时长超过30s），重启延迟将自动重置为0ms。
此外，如果我们希望只运行一次性脚本并且不希望重新启动脚本，我们可以使用之前介绍过的命令`pm2 start app.js --no-autorestart`启动来达到这一目的

## 集成
pm2 可以与`Docker`,`Heroku`,`Nginx`等集成，与nginx集成监听端口3001并且nginx将流量从端口80转到3001的示例
```json
upstream my_nodejs_upstream {
    server 127.0.0.1:3001;
}

server {
    listen 80;
    server_name my_nodejs_server;
    root /home/www/project_root;
    
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_max_temp_file_size 0;
        proxy_pass http://my_nodejs_upstream/;
        proxy_redirect off;
        proxy_read_timeout 240s;
    }
}
```
以上就是关于pm2的一些介绍，只要掌握以上的一些知识，相信我们可以很轻松的管理我们的应用，此外，关于pm2其他更全面的知识，可以去官网了解和查看。
