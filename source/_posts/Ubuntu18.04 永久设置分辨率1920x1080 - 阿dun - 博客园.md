---
title: Ubuntu18.04 永久设置分辨率1920x1080
date: 2020-02-20 10:44
categories: ['Linux']
tags: ['Linux']
---
###  起因

虚拟机(virtualBox)中设置 1920*1080 的分辨率后, 每次重启后都会回到默认,永久设置的方式如下

###  步骤:

  * ####  添加系统设置 

> sudo xrandr --newmode "1920x1080_60.00" 173.00 1920 2048 2248 2576 1080 1083
> 1088 1120 -hsync +vsync

> sudo xrandr --addmode Virtual1 "1920x1080_60.00"

此时系统设置里已经有 1920*1080 的分辨率选项

  * ####  打开开机启动配置 

> sudo vim /etc/profile

    * 按键 i 进入编辑模式, 按箭头下键把光标移动到文件最底部添加下面的内容 

> xrandr --newmode "1920x1080_60.00" 173.00 1920 2048 2248 2576 1080 1083 1088
> 1120 -hsync +vsync

> xrandr --addmode Virtual1 "1920x1080_60.00"

    * 按键 Esc 然后输入 :wq 回车, 保存退出. 

    * 重启 

> sudo reboot

