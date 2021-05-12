---
title:  部署Django云服务器 Gunicorn + Nginx 
date: 2021-01-03 10:10
categories: ['Django', 'Linux', 'Python']
tags: ['Django', 'python', 'Nginx']
---
#  工作流程

Django 自带的开发服务器性能太差，用到线上环境不合适。所以线上部署时，我们还要安装 ` Nginx ` 和 ` Gunicorn ` ，工作流程如下：

  * 客户端发来 http 请求，Nginx 作为直接对外的服务器接口，对 http 请求进行分析 
  * 如果是静态资源请求，则由Nginx自己处理（效率极高） 
  * 如果是动态资源请求，则把它转发给 Gunicorn 
  * Gunicorn 对请求进行预处理后，转发给 Django，最终完成资源的返回 

举例：去餐馆吃饭，Nginx 就是迎宾服务员，客人如果要酒水、果盘、菜单这些随手就能拿来的东西，迎宾自己就帮忙拿了；而 Gunicorn
是传菜服务员，Django 是厨师，他两一起满足客人对菜品的需求。

#  正式开始

##  远程连接

先连接到服务器:

  * **Windows** : 使用 **xshell** (简单快捷) 
  * **Mac** / **Linux** : 在终端配置 **ssh** 进行连接 (稍微复杂一些) 

####  Windows

在 Windows 环境开发的，推荐用 **XShell** 来作为远程连接的工具，简单高效。XShell 有 [ 学校及家庭版本
](https://www.netsarang.com/zh/free-for-home-school/) ，可以免费使用。

XShell 怎么使用就不多赘述了，随便一搜就能找到

> 基本就是把主机 IP、端口号（22）以及登录验证填好就能连接了。

####  Mac / Linux

  * 配置ssh密钥可以 [ 参考这里 ](https://blog.csdn.net/Richard__Ting/article/details/80012264)

  * 直接连接 

    * 输入 ` ssh root@ip ` 。（ ` ip ` 就是主机IP地址） 
    * 接下来会出现提示输入密码，然后输入你的Linux服务器的密码； 
    * 连接成功。 

##  代码部署

这里默认你已经装好了Python环境，没装好的话装好了再继续。

接下来就是要改一下 Django 的配置文件 ` settings.py ` ：

    
    
    settings.py
    
    # 关闭调试模式
    DEBUG = False
    
    # 允许的服务器（*代表全部都允许）
    ALLOWED_HOSTS = ['*']
    
    # 静态文件收集目录
    STATIC_ROOT = os.path.join(BASE_DIR, 'collected_static')
    

  * 部署时要关闭调试模式，避免安全性问题（此时 Django 就不再处理静态资源了）。 
  * ` ALLOWED_HOSTS ` 指明了允许访问的服务器名称或 IP，星号表示允许所有的请求。实际部署时请改成你的域名或 IP，比如 ` ALLOWED_HOSTS = ['.dusaiphoto.com', '127.0.0.1'] ` 。 
  * 项目中有很多静态文件，部署时需要找一个地方统一收集起来，也就是 ` STATIC_ROOT ` 指定的地址了。 

**环境一般是需要在服务器上重新生成的** ，因此我们需要把开发中用到的库列一个清单，以便在服务器上统一安装。在 **本地虚拟环境** 中输入：

    
    
    pip freeze > requirements.txt
    

项目中就多了个 ` requirements.txt ` 文件，里面记录了项目需要的库的清单。

将项目打包好，传入服务器中：

  * Windows下，xshell中输入 ` rz ` 选择文件进行传输（sz可以从服务器中回传） 

  * Mac/Linux下， ` scp 本地项目路径 用户名@ip:服务器接收路径 `

> scp /Users/aduner/project.zip root@81.70.151.147:/home/lighthouse/

在服务器中解压项目，并进入项目目录

接下来就是安装库、收集静态资源、数据迁移了：

    
    
    pip install -r requirements.txt 
    python manage.py collectstatic
    python manage.py migrate
    

代码部署基本就完成了，接下来配置 ` Nginx `

##  Nginx

为了防止系统太旧引起的各种麻烦，先升级一下库的版本：

    
    
    sudo apt-get update
    sudo apt-get upgrade
    

完成之后，接着安装nginx

    
    
    sudo apt-get install nginx
    

启动 nginx 服务：

    
    
    sudo nginx
    

打开浏览器，输入你的 **服务器公网 IP 地址**

Nginx 欢迎界面出现了，如果是centos系统可能会出现centos界面，但总之会得到一个反馈。

这个默认配置显然是不能用的，所以需要重新写 Nginx 的配置文件。进入 ` /etc/nginx/sites-available ` 目录，这里是定义
**Nginx 可用配置** 的地方。输入指令 ` sudo vi dusaiphoto.com ` 创建配置文件并打开 **vi 编辑器** ：

> centos系统下的nginx可能没有这个目录，具体可以 [ 参考这里
> ](https://www.cnblogs.com/aduner/p/14224858.html)
    
    
    cd /etc/nginx/sites-available
    sudo vim django_projact
    

在 ` django_projact ` 文件中写入：

    
    
    server {
      charset utf-8;
      listen 80;
      server_name xxx.xxx.xx.x;  # 改成你的 IP
    
      location /static {
        alias /home/sites/dusaiphoto.com/django_blog/static; # 根据自己的来
      }
    
      location /media {
        alias /home/sites/dusaiphoto.com/django_blog/media; # 根据自己的来
      }
    
      location / {
        proxy_set_header Host $host;
        proxy_pass http://unix:/tmp/xxx.xxx.xx.x.socket;  # 改成你的 IP
      }
    }
    

此配置会监听 80 端口（通常 http 请求的端口），监听的 IP 地址写你自己的 **服务器公网 IP** 。

配置中有3个规则：

  * 如果请求 static 路径则由 Nginx 转发到目录中寻找静态资源 
  * 如果请求 media 路径则由 Nginx 转发到目录中寻找媒体资源 
  * 其他请求则交给 Django 处理 

> 如果已经申请域名，就把配置中有 IP 的地方都修改为域名

写的只是 Nginx 的 **可用配置** ，所以还需要把这个配置文件链接到 **在用配置** 上去：

    
    
    ln -s /etc/nginx/sites-available/dusaiphoto.com /etc/nginx/sites-enabled
    

至此 Nginx 就配置好了，接下来搞定 ` Gunicorn ` 。

###  Gunicorn及测试

**先回到项目所在的目录** ，并且进入 **虚拟环境** ，然后输入：

    
    
    (env) ../django_blog_tutorial$ pip3 install gunicorn
    (env) ../django_blog_tutorial$ sudo service nginx reload
    (env) ../django_blog_tutorial$ gunicorn --bind unix:/tmp/118.31.35.48.socket my_blog.wsgi:application
    

这里的三个步骤分别是：

  * 安装 ` Gunicorn `
  * 重启 ` Nginx ` 服务 
  * 启动 ` Gunicorn `

> 启动 Gunicorn 也是一样，如果你已经有域名了，就把套接字中的 IP 地址换成域名；wsgi 字眼前面是项目的名称。另外 ` sudo
> service nginx reload ` 可替换成 ` sudo service nginx restart ` ，区别是 reload
> 只重载配置文件，restart 重启整个服务。

接下来用浏览器访问服务器试一下：

![](https://img2020.cnblogs.com/blog/1741852/202101/1741852-20210103100839181-971420996.png)

**大功告成**

