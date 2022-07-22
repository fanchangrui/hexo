---
title: js逆向之某度翻译
date: 2022-07-22 13:46:21  
categories: js逆向
tags:
- js逆向
- 反爬
---

## 前言
快速优雅地学会JS逆向，就需要从实战开始，接下来我会提供Base64加密的原网址以及接口参数，从实战中学习如何下断点、抠代码、本地运行等操作，此技术一般用于爬虫上，是一个爬虫程序猿进阶的必经之路。

## 实战信息
网址：https://fanyi.baidu.com/  
接口：https://fanyi.baidu.com/v2transapi  
逆向参数：
- sign:232427.485594  
- token:3dde9ef10b6f6ae310af38e6f1bd564f  

## 实战流程
### 1.抓包分析
首先，进入页面按F12，打开控制面板，调到Network板块后在翻译处写入需要翻译的信息(我这里输入的是“你好”)触发网络请求，打开请求面板查看该请求的具体信息。  
![这是图片](/img/722/1.png "Magic Gardens")  
General：请求信息  
- URL：请求API地址
- Method：请求方式GET/POST  
Request headers：请求头  
除了自定义请求头参数，其余Host、Origin、Referer一般为爬虫必须参数，Cookie看站点是否有对其校验，如果有特殊的自定义参数一般也为爬虫需要。   
- Acs-Token：自定义的校验参数
- Cookie：客户端缓存信息
- Host：域名
- Origin：来源信息
- Referer：防止跨站攻击  

### 2.查找加密参数
![这是图片](/img/722/2.png "Magic Gardens")   
 从上图中可以看到sign和token有加密嫌疑，所以目标就暂定为对这两个参数进行一个简单逆向。  

 ### 3.断点分类
 下断多种方法：  
 1.元素下断  
 2.事件下断  
 3.XHR下断  
 4.搜索下断  

 ### 4.学会快速下断
 这里主要展示XHR下断和搜索下断，在此比较好用
XHR断点：首先复制请求链接?前的部分路径，这里是/v2transapi,到Source下的XHR断点处下断，回车输入新翻译内容即可触发。  
![这是图片](/img/722/3.png "Magic Gardens")    
![这是图片](/img/722/4.png "Magic Gardens")    
可以看到断点断在了send()方法的调用处，在Call stack中查看函数方法的调用栈，目前代码可读性太低，我们需要对其进行格式化查看，就需要左下角的{}按钮。  
在调用栈中查看自己所需参数的作用域，首先在调用栈中找到加密后的参数位置，然后往前推，直到找到加密的方法，此处直接展示寻找结果。
![这是图片](/img/722/5.png "Magic Gardens")  
如上图，作用域的位置就看提示的参数数据即可，windows下如果使用Chrome可以用ctrl键触发，这里查看的是ajax()的方法栈，同时我们可以找到加密后的结果，鼠标放到函数参数里可以看到，我们也可以直接从Scope中查看然后返回源码找。      
![这是图片](/img/722/6.png "Magic Gardens")   
此处左右两侧都可以看到一个加密后的sign和token数据,就往前找调用的方法栈即可。  
![这是图片](/img/722/7.png "Magic Gardens")   
再往前一个方法栈就可以看到，headers里面必须有个"Acs-Token"的参数，data中的数据来源是b，往上看b是一个对象，请求内容在对象里，sign:x(n)｜token:window.common.token这两个参数是这样获取的。  
我们可以直接知道x(n)就是sign的加密函数,token存在window.common下。  
![这是图片](/img/722/8.png "Magic Gardens") 
 鼠标放置在x函数上会给一个跳转提示，点击index_61616b2.js有利于我们直接找到函数调用的方法，我们可以直接去抠代码本地运行,n鼠标放上去可以发现就是"你好",将断点打在8782行，重新键入更改断点作用域。  
![这是图片](/img/722/9.png "Magic Gardens")   
测试确认x(n)方法确实是sign获取的方法位置，进入函数。  
![这是图片](/img/722/10.png "Magic Gardens")    
此处可以直接抠到本地用Node运行，代码调用可以看到调用了两个作用域里的函数，所以对代码抠全，进行部分改写，此处直接粘代码。  
```
 
function a(r) {
    if (Array.isArray(r)) {
        for (var o = 0, t = Array(r.length); o < r.length; o++)
            t[o] = r[o];
        return t
    }
    return Array.from(r)
}
function n(r, o) {
    for (var t = 0; t < o.length - 2; t += 3) {
        var a = o.charAt(t + 2);
        a = a >= "a" ? a.charCodeAt(0) - 87 : Number(a),
        a = "+" === o.charAt(t + 1) ? r >>> a : r << a,
        r = "+" === o.charAt(t) ? r + a & 4294967295 : r ^ a
    }
    return r
}
function e(r) {
    var o = r.match(/[\uD800-\uDBFF][\uDC00-\uDFFF]/g);
    if (null === o) {
        var t = r.length;
        t > 30 && (r = "" + r.substr(0, 10) + r.substr(Math.floor(t / 2) - 5, 10) + r.substr(-10, 10))
    } else {
        for (var e = r.split(/[\uD800-\uDBFF][\uDC00-\uDFFF]/), C = 0, h = e.length, f = []; h > C; C++)
            "" !== e[C] && f.push.apply(f, a(e[C].split(""))),
            C !== h - 1 && f.push(o[C]);
        var g = f.length;
        g > 30 && (r = f.slice(0, 10).join("") + f.slice(Math.floor(g / 2) - 5, Math.floor(g / 2) + 5).join("") + f.slice(-10).join(""))
    }
    var u = void 0
      , l = "" + String.fromCharCode(103) + String.fromCharCode(116) + String.fromCharCode(107);
    u = null !== i ? i : (i = window[l] || "") || "";
    for (var d = u.split("."), m = Number(d[0]) || 0, s = Number(d[1]) || 0, S = [], c = 0, v = 0; v < r.length; v++) {
        var A = r.charCodeAt(v);
        128 > A ? S[c++] = A : (2048 > A ? S[c++] = A >> 6 | 192 : (55296 === (64512 & A) && v + 1 < r.length && 56320 === (64512 & r.charCodeAt(v + 1)) ? (A = 65536 + ((1023 & A) << 10) + (1023 & r.charCodeAt(++v)),
        S[c++] = A >> 18 | 240,
        S[c++] = A >> 12 & 63 | 128) : S[c++] = A >> 12 | 224,
        S[c++] = A >> 6 & 63 | 128),
        S[c++] = 63 & A | 128)
    }
    for (var p = m, F = "" + String.fromCharCode(43) + String.fromCharCode(45) + String.fromCharCode(97) + ("" + String.fromCharCode(94) + String.fromCharCode(43) + String.fromCharCode(54)), D = "" + String.fromCharCode(43) + String.fromCharCode(45) + String.fromCharCode(51) + ("" + String.fromCharCode(94) + String.fromCharCode(43) + String.fromCharCode(98)) + ("" + String.fromCharCode(43) + String.fromCharCode(45) + String.fromCharCode(102)), b = 0; b < S.length; b++)
        p += S[b],
        p = n(p, F);
    return p = n(p, D),
    p ^= s,
    0 > p && (p = (2147483647 & p) + 2147483648),
    p %= 1e6,
    p.toString() + "." + (p ^ m)
}
console.log(e('你好'))
```
调试发现报错(命令行输入node xx.js)：    
![这是图片](/img/722/11.png "Magic Gardens")  
因为本地环境没有window对象，因为l是一步计算的到为固定值，我们需要获取到window[l]的值，就先获取l的值，鼠标放在l上即可获取因为此算法为固定值，获取到l = "gtk"，所以此处的window[l] === window["gtk"],我们通过搜索ctrl+shift+f/Command+shift+f调出搜索面板，通过搜索(window["gtk"]｜window['gtk']｜window.gtk)这三个方法去查找，就看此处调用哪个，某度翻译用的window.gtk，是一个固定值。  
![这是图片](/img/722/12.png "Magic Gardens")  
直接抠值替换源码里的window[l]即可，调试发现i还是undefined,所以在变量上定义一个i的初始化方法即可。  
![这是图片](/img/722/13.png "Magic Gardens")  
再次测试调用，对比浏览器请求参数。  
![这是图片](/img/722/14.png "Magic Gardens")  
结果一致就说明获取成功了，获取到sign了还有个token值没有获取，这个又怎么获取呢。上面我们发现是window.common.token产生，通过搜索(window["common"]｜window['common']｜window.common)先找父节点值，这里用window['common']即可搜到，发现common是页面定义的变量，刷新页面发现token值写死我们这里就直接获取到了token值。  
![这是图片](/img/722/15.png "Magic Gardens")   

## js完整代码
```
var token = "3dde9ef10b6f6ae310af38e6f1bd564f"

function a(r) {
    if (Array.isArray(r)) {
        for (var o = 0, t = Array(r.length); o < r.length; o++)
            t[o] = r[o];
        return t
    }
    return Array.from(r)
}
function n(r, o) {
    for (var t = 0; t < o.length - 2; t += 3) {
        var a = o.charAt(t + 2);
        a = a >= "a" ? a.charCodeAt(0) - 87 : Number(a),
        a = "+" === o.charAt(t + 1) ? r >>> a : r << a,
        r = "+" === o.charAt(t) ? r + a & 4294967295 : r ^ a
    }
    return r
}
function e(r) {
    var o = r.match(/[\uD800-\uDBFF][\uDC00-\uDFFF]/g);
    if (null === o) {
        var t = r.length;
        t > 30 && (r = "" + r.substr(0, 10) + r.substr(Math.floor(t / 2) - 5, 10) + r.substr(-10, 10))
    } else {
        for (var e = r.split(/[\uD800-\uDBFF][\uDC00-\uDFFF]/), C = 0, h = e.length, f = []; h > C; C++)
            "" !== e[C] && f.push.apply(f, a(e[C].split(""))),
            C !== h - 1 && f.push(o[C]);
        var g = f.length;
        g > 30 && (r = f.slice(0, 10).join("") + f.slice(Math.floor(g / 2) - 5, Math.floor(g / 2) + 5).join("") + f.slice(-10).join(""))
    }
    var u = void 0
    , l = "" + String.fromCharCode(103) + String.fromCharCode(116) + String.fromCharCode(107);
    var i = null;
    u = null !== i ? i : (i = "320305.131321201" || "") || "";
    for (var d = u.split("."), m = Number(d[0]) || 0, s = Number(d[1]) || 0, S = [], c = 0, v = 0; v < r.length; v++) {
        var A = r.charCodeAt(v);
        128 > A ? S[c++] = A : (2048 > A ? S[c++] = A >> 6 | 192 : (55296 === (64512 & A) && v + 1 < r.length && 56320 === (64512 & r.charCodeAt(v + 1)) ? (A = 65536 + ((1023 & A) << 10) + (1023 & r.charCodeAt(++v)),
        S[c++] = A >> 18 | 240,
        S[c++] = A >> 12 & 63 | 128) : S[c++] = A >> 12 | 224,
        S[c++] = A >> 6 & 63 | 128),
        S[c++] = 63 & A | 128)
    }
    for (var p = m, F = "" + String.fromCharCode(43) + String.fromCharCode(45) + String.fromCharCode(97) + ("" + String.fromCharCode(94) + String.fromCharCode(43) + String.fromCharCode(54)), D = "" + String.fromCharCode(43) + String.fromCharCode(45) + String.fromCharCode(51) + ("" + String.fromCharCode(94) + String.fromCharCode(43) + String.fromCharCode(98)) + ("" + String.fromCharCode(43) + String.fromCharCode(45) + String.fromCharCode(102)), b = 0; b < S.length; b++)
        p += S[b],
        p = n(p, F);
    return p = n(p, D),
    p ^= s,
    0 > p && (p = (2147483647 & p) + 2147483648),
    p %= 1e6,
    p.toString() + "." + (p ^ m)
}
// console.log(e('你好'))
// console.log(token)

// 获取Sign
function getSign(str){
    return e(str)
}
// 获取Token
function getToken(){
    return token
}
// Node导出方法方式
module.exports = {
    getSign,
    getToken
}
```

 此文章仅做学习之用，请勿做违法违纪之事。



 