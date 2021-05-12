---
title: Git远程连接GitHub添加远程库
date: 2021-01-13 16:20
categories: []
tags: ['ssh', 'GitHub']
---
#  Git远程连接GitHub添加远程库

在这之前你必须有github的账户。

#  一.在github上添加一个仓库

New respository

![img](https:////upload-
images.jianshu.io/upload_images/4891612-c951d7c6f007d02f.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1022/format/webp)

写个名字，然后Creat repository

![img](https:////upload-
images.jianshu.io/upload_images/4891612-181922292136b0bc.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/596/format/webp)

这样你就有一个text的仓库了，但是里面是空的，接下来我们上传本地仓库到远端

![img](https:////upload-
images.jianshu.io/upload_images/4891612-5b7a3c4d7dc0ceee.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1028/format/webp)

#  二. 配置SSH

1.打开你的git ，输入:ssh

![img](https:////upload-
images.jianshu.io/upload_images/4891612-2f1e98170febc5aa.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/556/format/webp)

它打印出这个信息，说明配置好了。

2.接着输入 ssh-keygen -t rsa （主要是生成你跟github联系的秘钥key）

连续三个回车，key就生成了。就在红色箭头所指文件夹

![img](https:////upload-
images.jianshu.io/upload_images/4891612-523ce1305eb1122b.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/467/format/webp)

#  三.GitHub 上添加 SSH key

打开上面打印出文件夹所在位置，用文本编辑器打开 id_ras.pub文件  
把 id_ras.pub 公钥公布给github

复制

![img](https:////upload-
images.jianshu.io/upload_images/4891612-e9365d5b8166326f.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/793/format/webp)

打开github，

github上settings里面的SSH and GPG keys

![img](https:////upload-
images.jianshu.io/upload_images/4891612-eb097dc2af1cab8e.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1029/format/webp)

然后New SSH  
Title位置不需要填  
粘贴  
Add SSH key

![img](https:////upload-
images.jianshu.io/upload_images/4891612-d2ae533e68cf26cd.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/986/format/webp)

2017-05-16_171005.png

测试连接 ssh -T [ git@github.com
](https://link.jianshu.com?t=mailto:git@github.com)  
在第一次测试时会弹出警告，需要填写yes，然后回车

![img](https:////upload-
images.jianshu.io/upload_images/4891612-66e90571aa78287f.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/600/format/webp)

#  四.把我们本地仓库提交到github

##  方式一：先把仓库clone下来，然后在里面添加文件修改后在上传。

复制一下这个地址

![img](https:////upload-
images.jianshu.io/upload_images/4891612-b7a46e660536cbfd.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1046/format/webp)

随便一个文件夹下面，右键打开git

输入命令：git clone [ git@github.com
](https://link.jianshu.com?t=mailto:git@github.com) :LiKaiRabbit/text.git

下载完成。

![img](https:////upload-
images.jianshu.io/upload_images/4891612-725c17fca0c5f89f.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/445/format/webp)

打开这个文件夹后，再打开git

![img](https:////upload-
images.jianshu.io/upload_images/4891612-bdd5415db21ee6fb.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/269/format/webp)

然后把我们添加的text.md文件提交到本地仓库

![img](https:////upload-
images.jianshu.io/upload_images/4891612-4abbb909dca59f94.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/373/format/webp)

把本地仓库推送到远程仓库 ：git push origin master

![img](https:////upload-
images.jianshu.io/upload_images/4891612-b633d06cc3a42e06.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/413/format/webp)

我们远程仓库已经有这个文件了

![img](https:////upload-
images.jianshu.io/upload_images/4891612-282f573c4bc76cb9.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1045/format/webp)

##  方式二：本地仓库关联远程仓库（本地仓库与远程仓库没有冲突情况下）。

新建文件夹text2 ，然后git init 初始化仓库

![img](https:////upload-
images.jianshu.io/upload_images/4891612-256e1c9fd6bd0679.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/788/format/webp)

输入关联命令：git remote add origin [ git@github.com
](https://link.jianshu.com?t=mailto:git@github.com) :LiKaiRabbit/text.git

origin是你给这个远程仓库起的名字，单个惯例都这个叫，多个可以起其他的

[ git@github.com ](https://link.jianshu.com?t=mailto:git@github.com)
:LiKaiRabbit/text.git 仓库的地址

![img](https:////upload-
images.jianshu.io/upload_images/4891612-9c533207f5fcc77c.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/429/format/webp)

.

把远程仓库文件拉下来： git pull origin master

![img](https:////upload-
images.jianshu.io/upload_images/4891612-e56af021b5827a57.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/537/format/webp)

然后我们新建个文件提交上去  
1.新建一个a.md文件  
2.添加到本地仓库 git add .  
3.提交到本地仓库 git commit -m'a.md'  
4.git push origin master  
5.github上的远程仓库就有了

![img](https:////upload-
images.jianshu.io/upload_images/4891612-9ca51b610665411c.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/909/format/webp)

##  方式四：本地仓库关联远程仓库（本地仓库与远程仓库文件不一致，有冲突情况下）。

虽然关联了远程仓库，但是pull和push都是出现警告和错误。

![img](https:////upload-
images.jianshu.io/upload_images/4891612-1936290e17acc2e5.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/558/format/webp)

这时候需要合并冲突

命令: git pull origin master --allow-unrelated-histories

![img](https:////upload-
images.jianshu.io/upload_images/4891612-faa2b56b02abfb02.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/456/format/webp)

但是它会马上跳转到另一个界面：

![img](https:////upload-
images.jianshu.io/upload_images/4891612-c39e7b6e88ac1cb3.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/584/format/webp)

然后我们按什么键都不管用，界面被锁住了。

然而并不是，我们现在需要：  
1.按下ESC键  
2.输入 ：wq 注意冒号是英文状态下的  
3.按下回车 enter键

![img](https:////upload-
images.jianshu.io/upload_images/4891612-70324a7513993418.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/598/format/webp)

ok

合并文件拷贝下来了

作者：LiKaiRabbit  
链接： [ https://www.jianshu.com/p/5ad1ae0f7efd
](https://www.jianshu.com/p/5ad1ae0f7efd)  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

