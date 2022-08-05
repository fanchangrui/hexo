---
title: js逆向之模拟浏览器环境
date: 2022-08-05 13:30:17  
categories: js逆向
tags:
- js逆向
- 浏览器环境
---

## 前言
前段时间在js逆向学习过程中，除了一些简单网站的逆向没有使用或检测dom和bom，一般的网站都需要补全浏览器环境，为了更方便进行学习，我找了一些网上的教程结合自己实现了一个简单的浏览器环境框架。今天我就来讲一下开发的思路和具体怎么使用。

## 1 介绍
这个框架叫做catvm，参考了志远大佬的思路实现，完整的代码已经在github上开源，如果感兴趣可以访问我的github或者点击下面链接查看https://github.com/fanchangrui/catvm    
此项目已经被开源安全社区OSCS收录并检测安全，请大家放心食用。  
[![OSCS Status](https://www.oscs1024.com/platform/badge/fanchangrui/catvm.svg?size=small)](https://www.oscs1024.com/project/fanchangrui/catvm?ref=badge_small)

[![OSCS Status](https://www.oscs1024.com/platform/badge/fanchangrui/catvm.svg?size=large)](https://www.oscs1024.com/project/fanchangrui/catvm?ref=badge_large)

## 2 使用方法
首先我们先讲一下使用方法，如何实现和二次开发在第三节中讲到
- 把扣下来的代码放到code.js中
- 调试运行index.js
- 如果报错看错误补充检测的环境  
### 没错就是这么简单，如果你逆的网站监测不是那么严格的话，直接调试就可以出结果了，如果还检查了其他地方就要自己补全剩下的环境。

## 3 开发思路
这节我简单讲一下开发的思路和怎么进行补充环境来打造属于你自己的模拟环境，随着你逆向的网站越来越多，补充的环境也会越来越完善，相信你经过一段时间可以应对80%的环境检测。  

### 3.1 目录结构
src文件夹下分为catvm2、tools、code.js、index.js四个文件夹，其中catvm2是catvm的源码，tools是一些工具，code.js是你扣下来的代码，index.js是你的调试入口。tools只是一段方便自动获取浏览器环境的脚本，自己手补的可以不用管这个。  
核心的catvm2分为browser和tools，browser是具体的各个检测对象的实现代码，tools设计了一些框架缓存，封装的一些工具方法比如保护函数、代理、打印错误等。  
![这是图片](/img/805/1.png "Magic Gardens")

### 3.2 核心思想
因为node和浏览器环境等相似性，我们借用node环境等基础来实现，但是现在大多数网站都检测了node，所以我们引入vm2这个包来实现纯净的v8环境。  
dom和bom实际上就是在全局挂载了几个对象，我们以前在补充这些时可能直接声明了一个普通对象在加点属性就可以绕过检测，但是实际上这是不严谨的，有些网站会检测在原型上有没有，这也是js这个语言的原理。  
现在我们来简单实现如何在原型上添加对象：
已document这个对象为例，不熟悉js原型的建议先去学习一下再看
```
const Document =function Document()
{

}
Object.defineProperties(Document.prototype,{
    [Symbol.toStringTag]:{
        value:'Document',
        configurable:true,
    }
})
document = {}
document.__proto__ = Document.prototype
```
这样就实现了最简单的document对象，然后可以在这个对象下面加一下用到了属性和方法比如：
```
document.cookie = ''
document.referrer = location.href || ''
document.getElementById = function getElementById(id){
    return null
}
document.addEventListener = function addEventListener(type,listener,useCapture){   
}
document.createElement = function createElement(tagName){
        let tagname = tagName.toLowerCase() + ''
        if(catvm.memory.htmlElements[tagname] == undefined){
            debugger
        }
        return catvm.proxy(catvm.memory.htmlElements[tagname]())

}
```
但是实际上v8对这个对象做了很多限制，我们可以封装保护函数、代理等方法来加强,这个具体再tools讲。如果一个对象在浏览器中无法看具体方法，我们要在定义的时候抛出getter错误。具体每个对象的实现参考js官方文档mdn https://developer.mozilla.org/zh-CN/
```
throw new TypeError('Illegal constructor')
```

### 3.3 tools工具实现
- memory.js: 内存缓存，用来缓存一我们自己注入的一些对象，不污染全局环境，即使日后被检测了改一下名字也能逃过检测。
![这是图片](/img/805/2.png "Magic Gardens")  
- proxy.js:代理函数，给我们自己模拟的对象加上后就可以查看到这个对象下哪个属性或方法被检测到了，方便我们补环境。  
![这是图片](/img/805/3.png "Magic Gardens")  
- safefunction.js:保护函数，因为js每个对象的原型上都有toString方法，所以我们自己重写一个toString方法的来规避检测。尽量每一个函数都用safefunction保护一下。
![这是图片](/img/805/4.png "Magic Gardens")  
- node.js:整合工具文件，把上面几个文件合并成一个文件，方便我们使用。

### 3.4 注意点
- catcm.node.js和tools下的node.js都是整合的功能，每加入一个浏览器属性对象或者工具都要放入.node.js结尾的文件中，当然你也可以写个遍历函数循环读取加入。
- 本框架用的是es6模块语法，如果要使用node老版本的commonjs语法的请在pack.json中删除 "type"，这样自动使用commonjs语法进行编译。

## 后记
本框架只补充了浏览器的几个基础对象和开放接口，很有可能无法直接使用，建议有一定基础的开发者研究并不断完善，最后说一句：逆向虽好，但切记不要利用这个去做违法的事。