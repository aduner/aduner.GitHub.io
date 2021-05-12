---
title: 反爬虫，非标准的json格式解析
date: 2020-06-18 15:58
categories: ['Python', '爬虫']
tags: []
---
#  反爬虫，非标准的json格式解析

今天写爬虫的时候，发现有一些数据都是通过非标准的 json 格式进行传输的，标准的 json 我们可以将其转化为 Python
中的数据类型，进行查询，但对于一些类似于 json 但又非标准 json 格式的字符，就会比较头疼了。

例如下面这样

    
    
    hxbase_json1({
    list: [
    {Number: "1", StockNameLink: "stock_bg.aspx?code=688008&date=2019-12-31", industry: "澜起科技(688008)"}
    {Number: "2", StockNameLink: "stock_bg.aspx?code=600663&date=2019-12-31", industry: "陆家嘴(600663)"}
    {Number: "3", StockNameLink: "stock_bg.aspx?code=000006&date=2019-12-31", industry: "深振业A(000006)"}]})
    

看上去格式和 json 很像，但仔细观察会发现，这其中的 key 没有被单引号包裹起来，这就导致了它无法被简单的解析为字典类型。

自己用正则去重新构建要累死，在网上找到了一个包-demjson`

> pip install demjson
    
    
    import demjson
    text='hxbase_json1({list: [\
    {Number: "1", StockNameLink: "stock_bg.aspx?code=688008&date=2019-12-31", industry: "澜起科技(688008)"}\{Number: "2", StockNameLink: "stock_bg.aspx?code=600663&date=2019-12-31", industry: "陆家嘴(600663)"}\
    {Number: "3", StockNameLink: "stock_bg.aspx?code=000006&date=2019-12-31", industry: "深振业A(000006)"}]})'
    #去除多余的干扰字符，变成只有key没有引起来的假json
    t = text[13:-1] 
    data = demjson.decode(t)
    

