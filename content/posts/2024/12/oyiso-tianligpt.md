---
title: 为Oyiso主题引入TianliGPT摘要
description:
date: 2024-12-07 18:24:26
categories: [技术探索]
tags: [Oyiso,WordPress]
---

## 前言

在之前我在使用Hexo主题的时候，当时有些Hexo主题带了TianliGPT摘要功能，当时觉得有点意思并且还买了5万字额度，结果到换系统都没用完（无奈）。

最近这不是又换回WordPress Oyiso主题了嘛，看了下已经有WordPress主题引入TianliGPT的案例了，对照教程和相关代码照葫芦画瓢把摘要功能又给添上了，也算是解决我那剩下的几万字额度。

不过现在它已经支持到PostChat了，升级版的对话浮窗，更多功能，无限额度，不过18元/月劝退我了，什么时候我文章够多了再引入这个也不急（穷）

## 什么是TianliGPT

TianliGPT是一个基于GPT-3.5的文字摘要生成工具，你可以将需要提取摘要的文本内容发送给TianliGPT，稍等一会他就可以给你发送一个基于这段文本内容的摘要。

- 实时生成的摘要

- 自动生成，无需人工干预

- 一次生成，再次生成无需消耗key

- 包含文字审核过滤，适用于中国大陆

- 支持中国大陆访问

## 引入

### 安装插件与设置

首先前往WordPress PostChat插件的[Github仓库](https://github.com/zhheo/wordpress-plugin-postchat)下载插件并在后台上传安装。

![](https://file.101045700.xyz/2024/12/84bce8cfe0ab8276ba0f78f1b3db4e86.webp)

打开插件设置，在上面填入你的Key，获取Key和购买额度参考上面引导，如果只需要文章摘要可以关闭智能对话功能。

### 前端引入代码

![](https://file.101045700.xyz/2024/12/6a03b5d58939f9f070c358bc812c7070.webp)

打开Oyiso主题的自定义代码设置，把下方代码填入自定义HTML代码（头部）或者（底部）都可以，替换里面的tianliGPT\_key项为你的Key。

```
<link rel="stylesheet" href="https://ai.tianli0.top/static/public/tianli_gpt.min.css"><script>
let tianliGPT_postSelector = '#postchat_postcontent';
let tianliGPT_key = '1145141919810'; //填入你的Key
</script>
<script src="https://ai.tianli0.top/static/public/tianli_gpt.min.js"></script>
```

保存设置，然后Enjoy it.
