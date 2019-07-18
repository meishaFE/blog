---
title: 实用的 Echarts 开发技巧
date: 2019-07-18 14:00:00
author: Nathan
tags:
  - Echarts
  - JavaScript
---


## 实用的 Echarts 开发技巧


最近做了数据可视化有关的需求，使用的是 Echarts。第一次使用 Echarts，在大概阅读了配置文档 https://echarts.baidu.com/option.html#title
，查看了官方 demo 之后开始进行开发了。其中走了一些坑，也总结了一下经验，和大家分享一下。

在需求开发中使用的是柱状图和饼状图。本文不会讲解原理，也不会讲太多如何配置 chart，官方文档的说明已经很好了，所以在阅读本文前需求对 Echarts 有大概的认识。本文只会涉及到一些常见的开发场景，在讲不同的开发场景的时候会稍微讲一些配置知识。
对于文中涉及到的场景，如果你有更好的方案，欢迎讨论。

#### 1. 如何处理坐标轴文字在画布边上溢出的问题?
<hr>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/2kUZ8KC857ORHvarTiHA.UZFafmcHuTOTc*xewZoOpM!/b/dMUAAAAAAAAA&bo=8AU4BAAAAAADB.s!&rf=viewer_4)
<br>
刚遇到这个问题的时候第一反应是去文档里查找和坐标轴文字相关的配置，可是最后无功而返。想换种方式——文字换行，但即使换行也有可能出现溢出的问题，我们需要的是修改它的渲染方式。最后误打误撞看到了 grid 这个配置的 containLabel 属性。我们只需要设置这个属性为true，就可以完美避免文字溢出的问题。这里贴一下这个属性的配置说明：
<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/16pt1B1DH9m7gWdpMW03WmLCz.ZvA1M3UGWyXJ6EC30!/b/dAgBAAAAAAAA&bo=6AdmAgAAAAADB6k!&rf=viewer_4)
<br>
效果图：
<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/cPzjDLjCgnUziwhIZFeZ8e35IxagMXnCZiJvb9O50A0!/b/dLYAAAAAAAAA&bo=dAUaBAAAAAADB00!&rf=viewer_4)
<br>

#### 2. 如何将饼图的组成说明直接显示在 label，如何修改柱状图的默认提示框 ，如何使坐标轴的文字换行?
<hr>
图一：<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/VTY.qei9n4ckOlk5*uCNRy4wSghRrcz7W.mX.0neKwY!/b/dMMAAAAAAAAA&bo=lgmoAwAAAAADBxY!&rf=viewer_4)
<br>
图二：<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/LjrHQaVOxRdieK6eKh1H202Cv0xV6wdlVp3LQR4OFHc!/b/dL8AAAAAAAAA&bo=lgmgAwAAAAADBx4!&rf=viewer_4)
<br>
图三：<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/q*uoP812ioxF1VVVaecjxEC2z3t7LnHIKMyPU2eqg20!/b/dAgBAAAAAAAA&bo=jgmQAwAAAAADBzY!&rf=viewer_4)
<br>
三个截图的左边的chart的配置均来自 Echarts 官方 demo（三个图的数据做了修改，图三的坐标轴文字做了修改），三图的左边是 Echarts 的默认展示：默认的提示框，默认的组成说明，默认的坐标轴。如何来修改这些默认行为来达到右边的效果，眼看是三个问题，其实都是同类问题。

画布中的关于提示框或者显示的文字都可以通过 `formatter` 来修改默认的样式，针对不同场景，会有不同属性的 `formatter`，如图所示。
<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/DRknncpiqpBDtlmtpi2dZ4KFQy2tBB4H5Xm8CsCN4rg!/b/dLgAAAAAAAAA&bo=EAKAAgAAAAADB7I!&rf=viewer_4)
<br>
`formatter` 属性接受一个字符串或者回调函数。

    1. 字符串可以包含一些表示特定含义的字符串模版。说明：https://www.echartsjs.com/option.html#series-bar.tooltip.formatter
    2. 回调函数接受一系列参数。在我们修改默认展示的过程只需要用到第一个参数，它包含了默认展示相关的数据。不同类型的 formatter 会有不同的值。

现在我们根据上面三个情况来扩展需求：

1. 给饼图的组成说明文字后面显示对应的数值和所占比例；
2. 给堆图的提示框中的数字添加每一部分所占比例。
3. 当柱状图的坐标轴文字过长，文字需要换行；


##### 2-1. 给饼图的组成说明文字后面显示对应的数值和所占比例

默认情况下，在鼠标悬浮在 chart 上的时候，显示饼图对应的组成部分的相关数据。如何将这些信息直接显示在 `label` 上呢？如图一右所示：
这时需要我们修改 `label` 的配置，所以直接定位到饼图的 `label` 配置。 我们打印出 `formatter` 的回调函数的第一个参数，可以得到我们想要的信息。
<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/a3Q..MZPirCQou2yBiS.Hoo8S6xz3IM4xL7rhlmyAic!/b/dL4AAAAAAAAA&bo=hgNWAgAAAAADB*M!&rf=viewer_4)
<br>
在 formatter 的回调中返回我们想要显示的结果即可，代码地址：https://codepen.io/HongYU/pen/xooweN

##### 2-2. 给堆图的提示框中的数字添加每一部分所占比例

 前一个例子做了抛砖引玉，剩下来的效果就可以很容易做出来了。第二个场景，堆图的 tooltip 包含了这条bar中所有组成部分的数据，所以它的 `formatter` 回调函数的第一个参数会是一个数组。在原始的 tooltip 上添加信息，我们需要关注的属性有：
   
1. marker ： 图标的html
2. seriesName： 组成部分的名称
3. value： 组成部分的值

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/DoeRwWcOaOXsgSeDNW**cxl0nO88IV9trhol18z3w2w!/b/dL4AAAAAAAAA&bo=AAR.AwAAAAADB1s!&rf=viewer_4)
<br>
通过遍历数组算出一条 bar 的总值，然后算出对应的百分比。最后做字符串的拼接。代码地址：https://codepen.io/HongYU/pen/Bggjod

##### 2-3. 如何使坐标轴在长文本的时候换行？ 

定位问题：根据 “修改坐标轴的文本” 定位到 axisLabel，通过在它的 `formatter` 回调函数传一个拆分算法，然后将拆分结果用换行符号拼接即可。这种方式个人觉得比通过旋转x轴上的字体来显示长文要美观得多。这个大家可以自己来动手试试，如果需要修改 x 轴， 那么需要给 `xAxis.axisLabel` 配置 `formatter`， 同理，如果是 yAxis，就是 `yAxis.axisLabel`。

如何修改 Echarts 的默认显示就先介绍到这里。

总结： 

    1. 根据需要修改部分来定位它的配置信息，
    2. 通过 `formatter` 回调函数中的第一个参数里的信息，返回一个我们想要的结果。


#### 3. 如何给堆图中的 bar 上显示它的总数据
<hr>
效果图如下：
<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/5fvT9hr95Vwphzi9K5e*Q7v81UHvVdGc9pITcDK4g9s!/b/dFMBAAAAAAAA&bo=iAUWBAAAAAADB70!&rf=viewer_4)
<br>
Echarts 是可以通过配置来显示堆图中一条 bar 的每一节的数据（如下图），但是不能配置出在 bar 顶端显示整条 bar 的总数，那我们要怎么实现呢？
堆图中一条 bar 的每个组成部分代表着一个 serie，且他们的 stack 属性值是相同的。每个 serie 的显示可以根据它的 formatter 来修改。
<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/TYPS10K5HqFTsZ8Rgcu3vPrwMdXizZMG30gywnw*qwc!/b/dL8AAAAAAAAA&bo=SATgAgAAAAARB54!&rf=viewer_4)
<br>
思路：

1. 在 `series` 里面多添加一个 `serie`，这个 `serie` 的 data 长度和其他 serie 相等。
2. 新加的 `serie` 字段的数据为全为0，这样在图中就不会有显示新节点。
3. 隐藏其他 serie 的 label 显示，将新加的 `serie` 的 label 显示出来。
4. 这个 label 最初是0，因为我们给他传入的值是0，计算得到每条 bar 的总值，然后通过 `formatter` 将默认显示替换为这个总值。
<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/ecf5637SI7*YzOXAoBYsTYpvLAltZgWA8eZkG6YczM8!/b/dD4BAAAAAAAA&bo=0AEcAQAAAAADB.4!&rf=viewer_4)
<br>

代码地址：https://codepen.io/HongYU/pen/QXXjVq

如果是分组柱状体图呢，如何在bar的顶端显示该条 bar 的数值？
<br>

![image](http://m.qpic.cn/psb?/V13jYI8j0MYvwr/b*XbaCFPoL66f0.c2mk9GJvM4wsxGuUdpTAt1.ePWzM!/b/dMMAAAAAAAAA&bo=eAU4BAAAAAADB2M!&rf=viewer_4)
<br>

思路：在柱状图中， 相同 `stack` 属性值的 serie 会被放置在同一条 bar 堆砌在一起，不同的 `stack` 属性值的 serie 会被分组放置。
我们需要为每一种 `stack` 添加一个 “空” serie。其 `data` 数组中全是0；

在添加“空” `serie` 的时候，我们需要找到 和这个 `serie` 相同的 `stack` 属性值的 `serie`，然后在这个 `serie` 寻找对应的 `data`。

```
formatter: params => {
    let count = 0;
    chartData.forEach(d => {
        if (d.name !== 'totalLabel' && d.stack === stackName) {
            total = d.data[params.dataIndex];
        }
    });
    return count;
}
```

##### 延伸一：同时满足分组柱状图和堆图的 formatter 配置

其实稍微做一下改动就可以使在分组柱状图和堆图中使用相同的 formatter 配置：
将所有和这个新添加的 serie 有相同的 stack 属性值的 serie 进行相加。
```
formatter: params => {
    let total = 0;
    chartData.forEach(d => {
        if (d.name !== 'totalLabel' && d.stack === stackName) {
            total += d.data[params.dataIndex];
        }
    });
    return total;
}
```
代码地址：https://codepen.io/HongYU/pen/dBBRoQ

##### 延伸二： 希望可以点击 legend 筛选 chart 的数据显示，并且“总数”也是随着过滤后的数据来改变的。

在延伸一中，当用户点击legend后，bar 顶端的数字是不会发生改变的。我们需要监听 legend 的点击事件，来重新对它们进行计算。
点击 legend 的回调函数的第一个参数含有被点击的 legend 的信息和所有 legend 的状态。 我们就筛选出处于高亮的 legend 对应的 `serie`。
代码地址：https://codepen.io/HongYU/pen/KjjbVo

且这段代码同时满足分组柱状图和堆图。

我的总结内容到此结束了， 感谢阅读。

