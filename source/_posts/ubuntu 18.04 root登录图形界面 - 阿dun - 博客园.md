---
title: ubuntu 18.04 root登录图形界面
date: 2020-02-18 20:37
categories: ['Linux']
tags: ['Linux']
---
####  修改文件 

> vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf

  * ####  增加两行： 

> greeter-show-manual-login=true

> all-guest=false

  * ####  进入/etc/pam.d目录 

> cd /etc/pam.d

修改 ` gdm-autologin ` 和 ` gdm-password ` 文件

> vim gdm-autologin

注释掉 ` auth required pam_succeed_if.so user != root quiet_success `

> vim gdm-password

注释掉 ` auth required pam_succeed_if.so user != root quiet_success `

  * ####  修改/root/.profile文件 

> vim /root/.profile

将文件末尾的 ` mesg n || true ` 这一行修改成 ` tty -s&&mesg n || true `

  * ####  重启系统，输入root用户名和密码，登录系统 

