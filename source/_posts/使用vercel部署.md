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

  ![这是图片](/img/selectTemplate.png "Magic Gardens")

## 二、起步
打开vercel主页https://vercel.com/signup  
使用GitHub账号去关联vercel，后续代码提交到vercel可以自动触发部署  
出现授权页面，点击Authorize Vercel。
 
## 三、部署Hexo博客
vercel是最好用的静态站点托管平台，借助vercel平台，我们可以把博客静态文件部署到vercel上，不在使用  
GitHub pages托管，vercel比GitHub pages快多了。

- 选择一个vercel提供的模板部署，当然你也可以把代码提交到GitHub上，再去vercel选择即可

  ![这是图片](/img/select1.png "Magic Gardens")