---
title: Anaconda 常用命令大全
date: 2020-02-08 11:05
categories: ['Python']
tags: ['Anaconda', '环境']
---
###  帮助目录

> conda create -h

###  检查conda版本

> conda --version

###  升级当前版本的conda

> conda update conda

###  创建一个新环境

> conda create --name newname biopython

创建一个新的环境 newname，位置在Anaconda安装文件的/envs/newname

* * *

###  激活新环境

####  Linux，Mac:

> source activate snowflakes

####  Windows：

> activate snowflake

* * *

###  列出所有的环境

> conda info -e

* * *

###  切换环境(activate/deactivate)

####  Linux，OS X:

> source activate newname

####  Windows：

> activate newname

* * *

###  从当前工作环境的路径切换到系统根目录

####  Linux，Mac:

> source deactivate

####  Windows:

> deactivate

* * *

####  克隆环境

> conda create -n 被克隆环境 --clone 新环境

####  删除一个环境

> conda remove -n newname

####  安装一个新python

> conda create -n newname python=3

