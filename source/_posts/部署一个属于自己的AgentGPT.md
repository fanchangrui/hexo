---
title: 部署一个属于自己的AgentGPT
date: 2023-04-22 14:14:13
categories:  AI
tags:
- AI
- AgentGPT
---

前段时间Reworkd团队开源了AgentGPT，在github上迅速火爆起来，截止目前已经获得了16k的star，我研究了下这是一个基于ChatGPT能给定一个目标自主训练的AI项目，他的功能如下：
- 通过向量 DB 的长期记忆
- 通过 LangChain 的网页浏览功能
- 与网站和人的互动
- 通过文档 API 编写功能
- 节省代理运行
- 用户和身份验证
- 下限付费版本的 Stripe 集成

我们可以把这个项目部署在自己的电脑或服务器上，AgentGPT 允许您配置和部署自治 AI 代理。为您自己的自定义 AI 命名，让它开始任何可以想象的目标。它将尝试通过思考要完成的任务、执行任务并从结果中学习来实现目标。

## 安装AgentGPT
AgentGPT所用到的前端技术大概包括了Next.js,TailWindCSS和create-t3-app来快速创建Next应用。
这个项目有多种部署方式可以利用docker开始启动，我们今天来进行手动部署并修改一些创建自己的AgentGPT。

1、首先需要安装node18+的环境，可以在nvm中配置下；  
2、克隆仓库输入以下命令：  
  git clone https://github.com/reworkd/AgentGPT.git  
3、安装依赖项：
  npm install  
4、创建一个包含以下内容的.env文件并修改自己的内容：
```
# Deployment Environment:
NODE_ENV=development

# Next Auth config:
# Generate a secret with `openssl rand -base64 32`
# mac用系统自带的openssl命令生成随机字符，windows不清楚系统是否支持
NEXTAUTH_SECRET=changeme
NEXTAUTH_URL=http://localhost:3000
# 输入自己的数据库配置，我这里使用的mysql
DATABASE_URL=mysql://root:密码@localhost:3306/prisma

# Your open api key
# 输入你自己的openai的key
OPENAI_API_KEY=changeme
```
5、运行项目：
npx prisma db push  
npm run dev

现在你可以看到本地3000端口启动一个AgentGPT项目了。

## 修改代码和使用
在settings添加自己的key选择gpt-3.5或者gpt-4模型进行提问。
![这是图片](/img/230422/1.png "Magic Gardens") 
