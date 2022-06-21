---
title: scrapy框架爬取数据
date: 2022-06-21 16:17:16  
categories: python  
tags:
- python
- scrapy
- 爬虫
---

## scrapy框架介绍
- Scrapy是适用于Python的一个快速、高层次的屏幕抓取和web抓取框架，用于抓取web站点并从页面中提取结构化的数据。Scrapy用途广泛，可以用于数据挖掘、监测和自动化测试。  
- Scrapy吸引人的地方在于它是一个框架，任何人都可以根据需求方便的修改。它也提供了多种类型爬虫的基类，如BaseSpider、sitemap爬虫等，最新版本又提供了web2.0爬虫的支持。  

### 为什么使用scrapy？
scrapy框架，将网页采集的通用功能，集成到各个模块中，留出自己定义的部分，它将程序员从繁冗的流程式重复劳动中解放出来，简单的网页爬虫的重点，就集中在应对反爬，大规模爬取和高效稳定的爬取这几个方面。  
- scrapy 是异步的，可以灵活调节并发量
- 采取可读性更强的 xpath 代替正则，速度快
- 写 middleware,方便写一些统一的过滤器
- 同时在不同的 url 上爬行
- 支持 shell 方式，方便独立调试
- 通过管道的方式存入数据库，灵活，可保存为多种形式

## scrapy初步体验
### 安装配置
使用pip包管理工具安装scrapy，如果没有安装pip首先安装pip，因为我使用的是mac所以执行以下命令：
```
brew install pip
```
### 下载scrapy
因为m1pro自带python3所以使用pip3命令，如果环境是python2使用pip
```
pip3 install scrapy
```
### 新建项目
在开始爬取之前，必须创建一个新的Scrapy项目。进入自定义的项目目录中，运行下列命令：  
```
scrapy startproject mySpider
```
### 明确爬取的目标
我打算爬取豆瓣网的数据就在spider目录下创建douban.py的文件。

### 爬取数据
编写douban.py获取每个电影的标题、评分和主题
```
import scrapy
from scrapy import Selector,Request
from scrapy.http import HtmlResponse

from py3.items import MovieItem


class DoubanSpider(scrapy.Spider):
    name = 'douban'
    allowed_domains = ['movie.douban.com']
    start_urls = ['https://movie.douban.com/top250']

    def parse(self, response: HtmlResponse):
        sel = Selector(response)
        list_items = sel.css('#content > div > div.article > ol > li')
        for list_item in list_items:
            movie_item = MovieItem()
            movie_item['title'] = list_item.css('span.title::text').extract_first()
            movie_item['rank'] = list_item.css('span.rating_num::text').extract_first()
            movie_item['subject'] = list_item.css('span.inq::text').extract_first()
            yield movie_item

        hrefs_list = sel.css('div.paginator > a::attr(href)')
        for href in hrefs_list:
            url = response.urljoin(href.extract())
            yield Request(url=url)
```
在items.py中创建MovieItem类
```
import scrapy

class MovieItem(scrapy.Item):
    title = scrapy.Field()
    rank = scrapy.Field()
    subject = scrapy.Field()
```
### 把数据存到excel或者数据库中
scrapy提供了一个管道，可以将数据存到excel、csv文件或者数据库中。 
使用pip3下载并导入openpyxl，
在pipelines.py文件中写入以下代吗：
```
import openpyxl

class Py3Pipeline:

    def __init__(self):
        self.wb = openpyxl.Workbook()
        self.ws = self.wb.active
        self.ws.title = 'top250'
        self.ws.append(('标题', '评分', '主题'))

    def close_spider(self, spider):
        self.wb.save('爬虫.xlsx')

    def process_item(self, item, spider):
        title = item.get('title', '')
        rank = item.get('rank', '')
        subject = item.get('subject', '')
        self.ws.append((title, rank, subject))
        return item
```
### 注意事项
要在settings.py文件中配置管道，打开这段注释代吗：
```
ITEM_PIPELINES = {
   'py3.pipelines.Py3Pipeline': 300,
}
```

## 启动爬虫
运行爬虫命令：scrapy crawl douban
然后在第一句目录下就会有douban.csv和爬虫.xlsx文件，这两个文件就是爬虫结果。
![这是图片](/img/621/scrapy1.png "Magic Gardens")
![这是图片](/img/621/scrapy2.png "Magic Gardens")

## 额外配置
到这里一个最简单使用scrapy的爬虫读取和写入程序就完成了，但还有许多额外设置可以配置比如：
- 设置USER_AGENT防止网站认为是机器人
- 设置cookie来破解游戏网站需要登陆
- 编写自动化脚本来运行程序
- 下载中间件并在middlewares.py文件中配置
