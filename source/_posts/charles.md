---
title: 前端工具-Charles常用功能介绍
date: 2018-10-14 15:45:55
tags:
- 前端工具
---

## Charles之前端利器

工欲善其事必先利其器,作为一个前端工作者，抓包，代理，这些日常的操作应该是耳熟能详的；但是，如何能够便捷的达到目的呢，今天，就给大家介绍一下Charles这一前端利器。

<!-- more -->

### 下载安装

[安装包下载地址v4.2.1](http://cdn.meishakeji.com/static/charles/charles4.2.1.dmg)

下载后进行安装即可

### 破解
[下载破解jar包](http://cdn.meishakeji.com/static/charles/charles.jar)

下载完后在应用程序中找到charles图标

![charles logo](http://cdn.meishakeji.com/static/charles/logo.png)

选中右键选中显示包内容
然后进入到Contents>Java目录，将刚刚下载好的jar包复制黏贴过来，即完成了对charles的破解，就是这么简单

安装成功后，就可以打开心心念的charles了


### 开始抓包

第一步：勾选系统代理

![charles logo](http://cdn.meishakeji.com/static/charles/snip1.png)

第二步：设置可以代理的ip

![charles logo](http://cdn.meishakeji.com/static/charles/snip2.png)


![charles logo](http://cdn.meishakeji.com/static/charles/snip3.png)

ip范围设置0.0.0.0/0即可

通过以上的设置，我们即可抓取普通http的系统的请求包了

不过目前大部分的系统都是https类型的了，所以，要抓取这类型系统的请求包，还需要进行以下操作

1.安装Charles跟证书

定位到Help - SSL Proxying - Install Charles Root Certificate

![charles logo](http://cdn.meishakeji.com/static/charles/snip4.png)

2.下载完证书后点进去看看是否该证书被信任，如果没有被信任，需要手动改成信任

![charles tip](http://cdn.meishakeji.com/static/charles/snip5.png)

3.设置需要代理的域名

定位到 Proxy - SSL Proxying Settings

![charles tip](http://cdn.meishakeji.com/static/charles/snip7.png)

![charles tip](http://cdn.meishakeji.com/static/charles/snip8.png)

在这个界面即可添加你需要代理的域名

设置完以上的步骤后，就可以愉快的进行抓包了

![charles tip](http://cdn.meishakeji.com/static/charles/snip9.png)

### 使用Charles进行代理

在进行微信公众号开发的时候，经常会需要调试微信授权分享以及地理位置获取等一些功能，而这些功能又是建立在微信api初始化成功的基础上，而且使用微信的这些功能是需要在公众号上设置安全域名等一些信息的，这种情况下，使用本地IP来开发就无法满足这一情况了，所以，我们还是需要借助Charles来进行代理，帮助我们更方便的开发

找到Tools下的Map Remote,勾选中，并且点击add，填入你想代理的域名目标地址和实际IP地址，保存后即可；然后，手机访问代理域名地址，即可访问到实际ip的地址，从而达到欺骗浏览器，正常使用微信的功能的目的。

![charles tip](http://cdn.meishakeji.com/static/charles/snip10.png)

![charles tip](http://cdn.meishakeji.com/static/charles/snip11.png)

以上，即为charles的一些使用较为频繁的功能的介绍，除此之外，charles还有更多的功能，例如，模拟低网速，rewrite,breakpoints,重复发送请求，过滤网络请求，压力测试等等，这些功能都可以在我们平常的使用环节中去探究，总之，Charles作为前端开发的一大利器，希望大家能够真正的去了解并好好的使用。