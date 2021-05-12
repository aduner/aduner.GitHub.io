---
title: Python3中正则的贪婪匹配模式
date: 2020-01-29 20:09
categories: ['Python']
tags: ['python', '正则表达式']
---
##  什么是贪婪模式

  * 正则在进行匹配时,从开始位置查找最远的结束位置,这种模式称之为贪婪模式。 
  * 在进行HTML标签类似内容获取时,贪婪模式会导致整个内容的返回,需要使用非贪婪模式。 
  * 固定的书写规则 : ` .*? ` 这种方式就是非贪婪模式，或者说是惰性模式 
  * Python中默认使用贪婪模式 

##  例子

    
    
    >>> import re
    >>> str = '<div>---hello---</div><div>---world---</div>'
    
    >>> print(re.findall(r'<div>(.*?)</div>', str))  #非贪婪模式
    ['---hello---', '---world---']
    
    >>> print re.findall(r'<div>(.*)</div>', str)   #贪婪模式
    ['---hello---</div><div>---world---']
    

