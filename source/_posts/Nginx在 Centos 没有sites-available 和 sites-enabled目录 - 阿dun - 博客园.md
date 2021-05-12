---
title: Nginx在 Centos 没有sites-available 和 sites-enabled目录
date: 2021-01-03 10:00
categories: ['Django', 'Linux', 'Python']
tags: ['Nginx', 'Centos']
---
  * 创建 ` /etc/nginx/sites-available ` 和 ` /etc/nginx/sites-enabled `

  * ` vim /etc/nginx/nginx.conf ` 编辑 ` http ` 块内部添加如下内容 

    
    
    include /etc/nginx/sites-enabled/*;
    

