---
title: jQuery中bind()与on()绑定事件的区别
date: 2020-01-29 16:22
categories: ['前端']
tags: ['jQuery', 'JavaScript', '前端']
---
* * *

**` .on() ` 方法比 ` .bind() ` 方法多一个参数 ` selector ` **

**` .on() ` 的 ` selector ` 参数是筛选出调用 ` .on() ` 方法的dom元素的指定子元素，如： **

> **$("ul").on('click','li', function(){})**

**为动态添加的元素也能绑上指定事件**

* * *

**案例：**

    
    
    <body>
        <ul>
            <li>1</li>
            <li>2</li>
            <li>3</li>
            <li>4</li>
        </ul>
    
        <script>
            $("ul li").bind('click', function() {
                alert("boom!");
            });
            // $("ul li").on('click', function() {
            //     alert("boom!");
            // });
            // 和上面效果一样
            $("ul").append("<li>5</li>");
        </script>
    </body>
    
    

此时点击列表5并不会弹出消息框

给 ` .on() ` 添加 ` selector ` 参数 ` li ` :

    
    
    <body>
        <ul>
            <li>1</li>
            <li>2</li>
            <li>3</li>
            <li>4</li>
        </ul>
    
        <script>
            $("ul").bind('click','li', function() {
                alert("boom!");
            });
            $("ul").append("<li>5</li>");
        </script>
    </body>
    

此时点击列表5弹出消息框boom!

