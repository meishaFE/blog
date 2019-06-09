---
title: 24点小游戏算法
date: 2019-04-08 21:37:14
tags: 
- js
---

## 前言

最近做了一个小游戏叫24点元素，玩法就是随机4个1-13的数字，然后将它们加减乘除后等于24，每个数字必须用且只能用一次即可。 那么首先摆在面前的难点就是怎么随机这四个数字，能令他们通过不同的加减乘除方法后能等于24。

<!-- more -->

## 实现思路

先定义四张牌的数据结构，代码函数numWarp：
```javascript
{
    r: number, // 数字
    m: string // 运算的表达式（最后过滤出r === 24的时候，获取运算的方法）
}
```
- 假设只要两张牌，获取所有的计算结果的可能性，代码函数calm，
- 降维计算四张牌，保持a,b不变，组合c,d，代码函数allCalm，要保证a,b,c,d都计算到,则需要计算a,b,c,d; a,c,b,d; a,d,c,b; b,c,a,d; b,d,c,a; c,d,b,a这几种组合;
- 获取所有计算结果，匹配到等于24的数据，拉取出运算表达式

```javascript
    function numWarp(...num) {
        return num.map((a) => ({
            m: a+'',
            r: a
        }));
    }
    function calm(a,b) {
        var r = [
            {
                m: `(${a.m}+${b.m})`,
                r: a.r+b.r
            },
            {
                m: `(${a.m}-${b.m})`,
                r: a.r-b.r
            },
            {
                m: `(${b.m}-${a.m})`,
                r: b.r-a.r
            },
            {
                m: `(${a.m}*${b.m})`,
                r: a.r*b.r
            },
        ];
        a.r !== 0 && r.push({m: `(${b.m}/${a.m})`,r: b.r/a.r});
        b.r !==0 && r.push({m: `(${a.m}/${b.m})`,r: a.r/b.r});
        return r;
    }
    function allCalm(a,b,c,d,u) {
        var s = [], t = [];
        calm(a,b).forEach((i) => {
            s = s.concat(calm(i, c));
            t = t.concat(calm(i, d));
        });
        s.forEach((i) => {
            u = u.concat(calm(i, d));
        });
        t.forEach((i) => {
            u = u.concat(calm(i, c));
        });
        return u;
    }
    function get24(a,b,c,d) {
        [a,b,c,d]= dataWarp(a,b,c,d);
        allCalm(c,d,b,a,allCalm(b,d,a,c,allCalm(b,c,a,d,allCalm(a,d,b,c,allCalm(a,c,b,d,allCalm(a,b,c,d,[])))))).forEach((i) => {
            if (i.r === 24) {
                console.log(i.m);
            } else {
                console.log('没有算式组合能等于24')
            }
        });
    }
```
然后我们就可以做到随便给出4个数字就能算出是否有等于24的算式，也就可以反推到生成的数字可以组合成24点啦， 这样类似的实现方法有很多，比如采取暴力破解：穷举法，又或者是用javascript里面的eval函数也可以计算出表达式然后搞定它~

## 总结

个人理解能力与研究深度有限，故文章中有任何不妥，欢迎指出与交流！
