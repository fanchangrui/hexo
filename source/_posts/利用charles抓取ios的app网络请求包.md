---
title: 利用charles抓取ios的app网络请求包
date: 2023-01-04 13:30:19
categories:  抓包
tags:
- 抓包
- charles
---

## 为什么选择charles
相信大家在用windows抓包的时候经常用到fiddler进行抓包，fiddler虽然好用但是也有个弊端就是跨平台支持不好，当我换成mac的时候需要抓包就了解到charles这个跨windows、macos、linux三端的抓包工具，接下来是我在用这个软件时遇到的问题和解决方法。

## 1 下载charles并配置代理
到charles官网下载最新的dmg文件，查看当前连的局域网的ip地址，打开iphone设置连接同一个wifi，代理设置手动并填入mac的局域网ip，端口号默认为charles的8888。
![这是图片](/img/230104/1.jpeg "Magic Gardens") 

## 2 下载mac和iphone相关的ca证书
mac打开charles选择菜单栏的Help-SSL Proxying-Install Charles Root Certificate,如图所示
![这是图片](/img/230104/2.png "Magic Gardens") 
搜索charles双击打开证书设置加密套接字协议层为始终信任
![这是图片](/img/230104/3.png "Magic Gardens") 
然后在iphone上打开safari浏览器输入地址：chls.pro/ssl下载证书描述文件，打开设置安装并信任证书。

## 3 配置 SSL Proxying Setting
菜单栏点击Proxy-SSL Proxying Setting，在Host输入*，Port输入443来抓取https请求
![这是图片](/img/230104/4.png "Magic Gardens") 

## 4 总结or乱码
接下来我们可以打开iphone的app就行操作就会抓到https的请求包了，有可能会出现乱码的情况，原因可能是mac没下载ca证书或者没配置https代理。
![这是图片](/img/230104/5.png "Magic Gardens") 