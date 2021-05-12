---
title: Linux(Ubuntu) MySQL数据库安装与卸载
date: 2020-02-22 16:34
categories: ['Linux']
tags: []
---
#  安装

  * ####  首先检查系统中是否已经安装了MySQL 

> sudo netstat -tap | grep mysql

    * 没有显示已安装结果，则没有安装 

    * 如若已安装，可以选择删除。（删除方法放在下面） 

  * ####  如果没有安装，则安装MySQL. 

    * 在终端输入 

> sudo apt-get install mysql-server mysql-client

    * 在此安装过程中会让你设置root用户密码(管理MySQL数据库用户，非Linux系统用户)，按照要求输入即可。 

  * ####  测试安装是否成功： 

> sudo netstat -tap | grep mysql

基本上不报错就是按照成功了

  * ####  也可通过登录MySQL测试 

> mysql -uroot -p

    * 接下来会提示你输入密码,即刚刚设置的root密码 

    * 但有时候可能安装是没有提示设置root，可通过一下方式查看 

> sudo vim /etc/mysql/debian.cnf

      * 此档案开启后可以很快速的看到MySQL 预设的用户名称和密码， 
      * 该界面下看到，user和password 
      * 之后用此账户和密码登陆即可 

  
  

#  修改远程访问

  * ####  第一步：修改配置文件的端口绑定 

打开的目录可能会根据MySQL的版本稍有不同，可以先尝试打开 ` /etc/mysql/my.cnf `
这个配置文件，若该文件不存在或文件内容为空，则尝试下面的文件路径。

> sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

在下面行的开头加上#，注释掉该行，然后保存退出vim：

> bind-address = 127.0.0.1

  * ####  第二步：修改访问权限 

进入mysql,输入如下命令，输入密码，进入mysql命令行

> mysql -u root -p

授权root用户访问权限，并刷新权限，此处的root可用其它MySQL用户替换

> grant all privileges on _._ to root@"%" identified by "pwd" with grant
> option;

> flush privileges;

> quit;

  * ####  第三步：重启mysql服务 

> service mysql restart

  
  

#  卸载

  * 查看MySQL的依赖项： ` dpkg --list|grep mysql `
  * 卸载： ` sudo apt-get remove mysql-common `
  * 卸载： ` sudo apt-get autoremove --purge mysql-server-5.0 `
  * 清除残留数据： ` dpkg -l|grep ^rc|awk '{print$2}'|>sudo xargs dpkg -P `
  * 再次查看剩余依赖项： ` dpkg --list|grep mysql `
  * 继续删： ` sudo apt-get autoremove --purge mysql-apt-config `
  * 删光结束~ 

