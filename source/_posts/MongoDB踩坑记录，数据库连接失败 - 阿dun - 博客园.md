---
title: MongoDB踩坑记录，数据库连接失败
date: 2021-03-10 09:10
categories: ['MongoDB']
tags: ['数据库', 'mongodb']
---
#  mac下mongo退出时候没有正常退出，之后无法正常连接数据库

**原因**

是mongo非正常退出不会释放锁，所以下次就进不去了

**目前找到的解决方案**

  * 查看mongo进程，全kill掉 

> ps -ef|grep mongo
>
> kill 23415 # 别跟我一样看自己的啥样

  * 删除mongo的lock 

我的是存到了这个地方

**/usr/local/mongodb/data/db**

删除这个目录下的.lock

  * 重新启动mongo服务 

> sudo mongod -dbpath=/usr/local/mongodb/data/db &

  * 登陆mongo 

> mongo

  * 解决 

