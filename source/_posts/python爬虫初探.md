---
title: python爬虫初探
date: 2022-06-10 14:17:31  
categories: python 
tags:
- python
- 爬虫
---

## 前言
最近在京东抢护肤品的试用装，发现一个一个手动点太麻烦了，所以想写个脚本来自动抢这些东西，顺便把爬虫的基础知识写下来。所谓生命不停，学习不止。
### 网络爬虫是什么
网络爬虫又称网络蜘蛛、网络机器人，它是一种按照一定的规则自动浏览、检索网页信息的程序或者脚本。网络爬虫能够自动请求网页，并将所需要的数据抓取下来。通过对抓取的数据进行处理，从而提取出有价值的信息。  
### 为什么用Python做爬虫
首先应该明确，不止 Python 这一种语言可以做爬虫，诸如 PHP、Java、C/C++ 都可以用来写爬虫程序，但是相比较而言 Python 做爬虫是最简单的。下面对它们的优劣势做简单对比：  
- PHP：对多线程、异步支持不是很好，并发处理能力较弱；Java 也经常用来写爬虫程序，但是 Java 语言本身很笨重，代码量很大，因此它对于初学者而言，入门的门槛较高；C/C++ 运行效率虽然很高，但是学习和开发成本高。写一个小型的爬虫程序就可能花费很长的时间。
- 而 Python 语言，其语法优美、代码简洁、开发效率高、支持多个爬虫模块，比如 urllib、requests、Bs4 等。Python 的请求模块和解析模块丰富成熟，并且还提供了强大的 Scrapy 框架，让编写爬虫程序变得更为简单。因此使用 Python 编写爬虫程序是个非常不错的选择。  
### 编写爬虫的流程
- 先由 urllib 模块的 request 方法打开 URL 得到网页 HTML 对象。
- 使用浏览器打开网页源代码分析网页结构以及元素节点
- 通过 Beautiful Soup 或则正则表达式提取数据。
- 存储数据到本地磁盘或数据库。
  当然也不局限于上述一种流程。编写爬虫程序，需要您具备较好的 Python 编程功底，这样在编写的过程中您才会得心应手。爬虫程序需要尽量伪装成人访问网站的样子，而非机器访问，否则就会被网站的反爬策略限制，甚至直接封杀 IP，相关知识会在后续内容介绍。  

## 第一个python程序
本节编写一个最简单的爬虫程序，作为学习 Python 爬虫前的开胃小菜。  
下面使用 Python 内置的 urllib 库获取网页的 html 信息。注意，urllib 库属于 Python 的标准库模块，无须单独安装，它是 Python 爬虫的常用模块。 
### 获取网页html信息
获取响应对象  
向百度发起请求，获取百度首页的 HTML 信息，代码如下  
```
#导包,发起请求使用urllib库的request请求模块
import urllib.request
# urlopen()向URL发请求,返回响应对象,注意url必须完整
response=urllib.request.urlopen('http://www.baidu.com/')
print(response)
```
上述代码会返回百度首页的响应对象， 其中 urlopen() 表示打开一个网页地址。注意：请求的 url 必须带有 http 或者 https 传输协议。  
输出结果，如下所示：
```
<http.client.HTTPResponse object at 0x032F0F90>
```
输出HTML信息  
在上述代码的基础上继续编写如下代码：  
```
#提取响应内容
html = response.read().decode('utf-8')
#打印响应内容
print(html)
```
输出结果如下，由于篇幅过长，此处只做了简单显示：  
```
<!DOCTYPE html><!--STATUS OK--> <html><head><meta http-equiv="Content-Type" content="text/html;charset=utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"><meta content="always" name="referrer"><meta name="theme-color" content="#2932e1"><meta name="description" content="全球最大的中文搜索引擎、致力于让网民更便捷地获取信息，找到...">...</html>
```
通过调用 response 响应对象的 read() 方法提取 HTML 信息，该方法返回的结果是字节串类型(bytes)，因此需要使用 decode() 转换为字符串。程序完整的代码程序如下：  
```
import urllib.request
# urlopen()向URL发请求,返回响应对象
response=urllib.request.urlopen('http://www.baidu.com/')
# 提取响应内容
html = response.read().decode('utf-8')
# 打印响应内容
print(html)
```
这样我们就实现了最简单的一个爬虫程序

## Python爬虫抓取网页
现在我们可以真实写一个实战程序来抓取网页并保存到自己的计算机。

### 导入所需模块
本节内容使用 urllib 库来编写爬虫，下面导入程序所用模块：  
from urllib import request  
from urllib import parse  

### 拼接URL地址
定义 URL 变量，拼接 url 地址。代码如下所示：  
```
url = 'http://www.baidu.com/s?wd={}'
#想要搜索的内容
word = input('请输入搜索内容:')
params = parse.quote(word)
full_url = url.format(params)
```

### 向URL发送请求
发送请求主要分为以下几个步骤：
- 创建请求对象-Request
- 获取响应对象-urlopen
- 获取响应内容-read
代码如下所示：  
```
#重构请求头
headers = {'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.64 Safari/537.36'}
#创建请求对应
req = request.Request(url=full_url,headers=headers)
#获取响应对象
res = request.urlopen(req)
#获取响应内容
html = res.read().decode("utf-8")
```
请求头带上自己电脑的浏览器信息，这样会识别为浏览器访问防止被反爬。
### 保存为本地文件
把爬取的照片保存至本地，此处需要使用 Python 编程的文件 IO 操作，代码如下：
```
filename = word + '.html'
with open(filename,'w', encoding='utf-8') as f:
    f.write(html)
```
运行输入hexo会在本地生成hexo.html的文件，该文件内容为爬取的网页内容。
![这是图片](/img/610/pc.png "Magic Gardens")