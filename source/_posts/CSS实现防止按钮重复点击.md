---
title: CSS实现防止按钮重复点击
date: 2022-11-23 14:09:55
categories:  CSS
tags:
- CSS
- click
---

## 前言
我们在工作中经常遇到需要按钮做防止重复点击的需求，以前我们实现这个节流的功能会用节流函数或者使用第三方库来快速实现，但是现在项目中一般使用按钮loading或者全局loading来实现这个功能，这也让我们停止思考了还可以用其他方式来实现防止停止点击，实际上使用css的几个属性就能快速做到这个效果。

## 实现思路
首先我们要思考下css怎么触发类似js的click事件呢？我们日常生活中可能用到的伪类可能是hover这种，但是实际上css有个active伪类可以在按钮点击的时候更新样式，然后依靠什么属性来禁止点击呢，答案是pointer-events这个属性，当它为none时可以使元素不成为鼠标点击事件，最后还有一个问题是怎么控制间隔时间呢，我想到了利用动画animation。

## 具体实现方式
我们可以利用active伪类来实现点击按钮时触发动画，然后设置动画的起点和终点的点击分别为可以点击auto和不可点击none，具体实现代码如下：    
```
button{
  animation: throttle 2s step-end forwards;
}
button:active{
  animation: none;
}
@keyframes throttle {
  from {
    pointer-events: none;
  }
  to {
    pointer-events: all;
  }
}
```

## 总结
虽然我们工作中可能不会用这个方式来防止重复点击，但这对我们理解css可能会有一些帮助，也在我们烦杂的生活中增添一丝乐趣。

