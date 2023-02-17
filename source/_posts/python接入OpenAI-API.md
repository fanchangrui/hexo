---
title: 预言成功！new必应来了！以及python接入OpenAI API
date: 2023-02-17 15:14:32
categories:  AI
tags:
- OpenAI
- python
---

两个月前我分享了chatgpt的注册和使用后预言了是否能颠覆Google后，没想到刚刚过去60天，微软就推出了整合chatgpt和必应的new必应搜索，这一措施让微软一洗颓势，从今后这条可能最重要的赛道上领先于Google和其他任何公司。本来今天我想聊一下新必应的应用，结果由于申请的候补一直没通过，所以就先分享一下如何在python接入openai的api，今后有很多应用会基于这个进行开发。

## 生成api key
首先注册openai就没必要说了，如果没有注册的可以看我12月8号发的文章，这里提醒下现在注册和使用条件变严格了，有些节点或者手机号并不能成功，需要大家多寻找可用的节点和手机号。
第一步我们访问https://platform.openai.com/account/api-keys 这个网站，然后点击create new secret key生成密匙，
![这是图片](/img/230217/1.png "Magic Gardens") 
这里要注意生成的key下次就看不了，我建议复制保存到电脑上，要不然下次又要重新生成一个key。

## python接入api
openai官方给了node和python两种方式接入，这里为了演示方便，选择用python来接入，
第一步安装openai包
```
pip install openai
```

## 文本类型
openai官方有很多模型来训练，我们熟知的chatgpt就是使用gpt3.5模型来训练，我们这里选择三种常见的模型文本、代码和图片来进行演示，这里我们可以用中文和英文来进行询问，但是中文返回结果要进行序列化所以我们暂时使用英文。
```
import openai
openai.api_key = "你的key"
prompt ="""
Does chatgpt belong to openai? Yes or no
"""
response = openai.Completion.create( model="text-davinci-003", prompt=prompt, max_tokens=100, temperature=0  )
print(response)
```
下面是返回的部分内容：
![这是图片](/img/230217/2.png "Magic Gardens") 
这里的text就是返回的结果，但是不知道为什么他说chatgpt不属于openai😛  
这里我解释下发送create的参数：
model：使用的模型类别
prompt：你要问的问题
max_token：完成的最大token数量
temperature：回答问题的程度，越接近0越标准，接近于1越开放

## 代码类型
代码类型使用codex模型来查询，这里我们输入问题是用js写一个验证手机号的正则表达式
```
import openai
openai.api_key = "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"
prompt ="Write a regular expression in javascript that validates the phone number"
response = openai.Completion.create(
  model="code-davinci-002",
  prompt=prompt,
  temperature=0,
  max_tokens=256,
  top_p=1,
  frequency_penalty=0,
  presence_penalty=0)
print(response)
```
下面是返回结果：
      "text": ".\n\nThe phone number should be in the following format:\n\n(123) 456-7890\n\nThe number should start with an opening parenthesis, followed by 3 digits, a closing parenthesis, a space, 3 digits, a hyphen, and 4 digits.\n\nThe phone number can optionally have an extension, which consists of a space, the letter x, and 3 or more digits.\n\nExamples:\n\n(123) 456-7890\n\n(123) 456-7890 x1234\n\n(123) 456-7890 ext1234\n\n(123) 456-7890 ext.1234\n\n*/\n\n// Write your code here\n\nlet phoneNumber = /^\\(\\d{3}\\)\\s\\d{3}-\\d{4}(?:\\sx\\d{3,})?$/;\n\n// Tests\n\nfor (let str of [\"(123) 456-7890\", \"(123) 456-7890 x1234\", \"(123) 456-7890 ext1234\", \"(123) 456-7890 ext.1234\"]) {\n  if"

## 图片类型
图片类型我们使用dall-e模型来生成，我输入的内容是一只猫和一条狗互相玩耍的照片，输入限制条件越多越容易得到想要的结果。
```
import openai
openai.api_key = "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"
response = openai.Image.create(
  prompt="A cat and a dog play with each other",
  n=1,
  size="1024x1024"
)
image_url = response['data'][0]['url']
print(image_url)
```
下面是我得到的图片：
![这是图片](/img/230217/3.png "Magic Gardens") 
参数size可以调整你想要图片的大小真是太方便了

## 总结
以上只是openai的api最简单实用方式，如果想要深入了解可以访问官方文档 https://platform.openai.com/docs/api-reference/edits 来进行学习，当然别人也不是做慈善的，每个人有一定的免费额度，超过了就要进行付费，当然我们平时学习这点额度肯定也是够的，相信过不了多久国内大厂也会有相关的api开放给我们使用，我的建议是好好学习这方面的知识毕竟很有可能是下一个风口。