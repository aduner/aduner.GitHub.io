---
title: Django中render和render_to_response的区别
date: 2020-08-22 17:20
categories: ['Django', 'Python']
tags: []
---
###  导入

    
    
    from django.shortcuts import render, render_to_response
    

###  作用

两者均是用来展示模板页面的。

###  参数区别

最明显的一个，render的第一个参数是request,后面的参数则和render_to_response相同

###  区别

由于传入参数的不同，造成了一个最直接的问题：  
能否在模板中使用request的属性，例如session等。  
所以，如果你要在模板中使用request的属性，请务必使用render。否则模板渲染时是得不到相应的request变量值的。

