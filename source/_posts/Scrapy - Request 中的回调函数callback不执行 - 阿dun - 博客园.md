---
title: Scrapy - Request 中的回调函数callback不执行
date: 2020-04-26 21:08
categories: ['Python', '爬虫']
tags: ['爬虫', 'python']
---
###  回调函数callback不执行

大概率是被过滤了  
两种方法:

  * 在 ` allowed_domains ` 中加入目标url 
  * 在 ` scrapy.Request() ` 函数中将参数 ` dont_filter=True ` 设置为 ` True `

