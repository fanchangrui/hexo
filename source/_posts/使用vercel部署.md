---
title: 使用vercel部署
date: 2022-04-21 11:15:33
tags:
---

## 一、介绍一下vercel
- vercel类似于github page，但远比github page强大，速度也快得多得多，而且将Github授权给vercel后，可
  可以达到最优雅的发布体验，只需将代码轻轻一推，项目就自动更新部署了。
- vercel还支持部署serverless接口。那代表着，其不仅仅可以部署静态网站，甚至可以部署动态网站，而这些功能
  能，统统都是免费的。
- vercel还支持自动配置https，不用自己去FreeSSL申请证书，更是省去了一大堆证书的配置
- vercel目前的部署模板有31种之多

  ![这是图片](/img/421/selectTemplate.png "Magic Gardens")
<!--more-->

## 二、起步
- 打开vercel主页https://vercel.com/signup  
- 使用GitHub账号去关联vercel，后续代码提交到vercel可以自动触发部署  
- 出现授权页面，点击Authorize Vercel。
 
## 三、部署Hexo博客
vercel是最好用的静态站点托管平台，借助vercel平台，我们可以把博客静态文件部署到vercel上，不在使用  
GitHub pages托管，vercel比GitHub pages快多了。

- 选择一个vercel提供的模板部署，当然你也可以把代码提交到GitHub上，再去vercel选择即可

  ![这是图片](/img/421/select1.png "Magic Gardens")
- 创建一个GitHub项目，代码会自动在GitHub账号上创建
  ![这是图片](/img/421/creat1.png "Magic Gardens")
- 创建完成后，等待vercel构建
  ![这是图片](/img/421/deploy1.png "Magic Gardens")
- 创建成功后自动跳到主页
  ![这是图片](/img/421/success1.png "Magic Gardens")
- 点击visit即可访问创建好的服务，vercel会分配给我们一个默认的域名，当然你也可以自定义修改。
  ![这是图片](/img/421/index1.png "Magic Gardens")
- 你也可以从Github选择代码来创建项目
  ![这是图片](/img/421/import1.png "Magic Gardens")
- 导入GitHub账号上的项目
  ![这是图片](/img/421/import2.png "Magic Gardens")

部署vue、react等前端项目过程也类似，这里不再演示

## 四、部署serverless接口
- 因为暂时没有部署动态网站的需求所以没有使用，具体使用方法请自行百度。