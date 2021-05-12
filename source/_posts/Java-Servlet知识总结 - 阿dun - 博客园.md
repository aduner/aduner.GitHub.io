---
title: Java-Servlet知识总结
date: 2021-01-21 19:39
categories: ['Java']
tags: ['后端', 'web', 'Java']
---
#  Servlet概述

##  为什么要学习Servlet

**Java Web的演变过程大概可以分为4个阶段：**

  * Servlet + jdbc + jsp 
  * Spring + Struts2+ Hibernate（SSH） 
  * Spring + SpringMVC + Mybatis（SSM） 
  * 微服务阶段 

前两个阶段基本上可以说是历史了，当今Spring家族一统天下。

现在实际开发中很少直接使用Servlet了，但是各个框架的底层还是大量使用了Servlet，学习Servlet对后续各个框架的理解和学习都很有帮助。

##  什么是 Servlet

Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP
服务器上的数据库或应用程序之间的中间层。

Servlet其实就是一个 **遵循Servlet开发的java类** 。Serlvet是 **由服务器调用的** ，运行在服务器端。

Servlet带给我们最大的作用就是能够 **处理浏览器带来HTTP请求，并返回一个响应给浏览器，从而实现浏览器和服务器的交互** 。

##  工作流程

  * Tomcat将浏览器提交的请求封装成HttpServletRequest对象，同时将输出流封装成HttpServletResponse对象 

  * Tomcat把request、response作为参数，调用Servlet的相应方法，例如doGet(request, response)等 

  * Servlet中主要处理业务逻辑 

* * *

#  生命周期

在 Web 容器中，Servlet 主要经历 4 个阶段，如下图：  
[ ![Servlet 生命周期](https://upload-
images.jianshu.io/upload_images/7896890-2599bf65a828350d.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1240) ](https://upload-
images.jianshu.io/upload_images/7896890-2599bf65a828350d.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1240)

  1. **加载Servlet** 。当Tomcat第一次访问Servlet的时候， **Tomcat会负责创建Servlet的实例**
  2. **初始化** 。当Servlet被实例化后，Tomcat会 **调用` init() ` 方法初始化这个对象 **
  3. **处理服务** 。当浏览器 **访问Servlet** 的时候，Servlet 会 **调用` service() ` 方法处理请求 **
  4. **销毁** 。当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动 **调用` destroy() ` 方法 ** ， **让该实例释放掉所占的资源** 。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁 
  5. **卸载** 。当Servlet调用完 ` destroy() ` 方法后，等待垃圾回收。 

如果 **有需要再次使用这个Servlet，会重新调用` init() ` 方法进行初始化操作 ** 。

只要访问Servlet， ` service() ` 就会被调用。 ` init() ` 只有第一次访问Servlet的时候才会被调用。

` destroy() ` 只有在Tomcat关闭的时候才会被调用。

* * *

#  处理请求的方法

**Servlet** 即实现了 **Servlet 接口** 的类，实现 **Servlet 接口** 的时候，需要实现5个方法

    
    
    public interface Servlet {
        void init(ServletConfig var1) throws ServletException;
    
        ServletConfig getServletConfig();
    
        void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
    
        String getServletInfo();
    
        void destroy();
    }
    

为了方便使用可以直接 **继承 HttpServlet** 类，该类已经默认实现了 Servlet 接口中的所有方法。

在编写 Servlet 的时候，只需要重写你需要的方法就好了。

并且该类还在原有 Servlet 接口上添加了一些与 HTTP 协议处理相关的方法。

  * Servlet 处理请求的方法一共有三种： 
    * 实现 ` service() ` 方法 
    * 重写 ` doGet() `
    * 重写 ` doPost() `

#  HttpServletRequest 和 HttpServletResponse

对于 **每次访问** 请求， **Servlet引擎** 都会创建一个 **新的HttpServletRequest请求对象** 和一个
**新的HttpServletResponse响应对象** ，即 request 和 response 对象。

###  HttpServletRequest 常用方法

  * ` String getContextPath() `   
获取上下文路径

  * ` String getHeader(String headName) `   
根据指定的请求头获取对应的请求头的值.

  * ` String getRequestURI() `   
返回当期请求的资源名称. 上下文路径/资源名

  * ` StringBuffer getRequestURL() `   
返回浏览器地址栏的内容

  * ` String getRemoteAddr() `   
返回请求服务器的客户端的IP

####  获取请求参数的方法：

  * ` String getParameter(String name) `   
根据参数名称,获取对应参数的值.

  * ` String[] getParameterValues(String name) `   
根据参数名称,获取该参数的多个值.

  * ` Enumeration getParameterNames() `   
获取所有请求参数的名字

  * ` Map<String,String[]> getParameterMap() `   
返回请求参数组成的Map集合.

###  HttpServletResponse 常用方法

  * ` OutputStream getOutputStream(): `   
获取字节输出流: **文件下载**

  * ` Writer getWriter() `   
获取字符输出流: **输出内容**

  * ` resp.setContentType("text/html;charset=utf-8") `

设置文件输出的编码格式和内容类型

  * ` resp.sendRedirect() `

302重定向，临时跳转

301要使用另外的手段

#  Servlet 是单例的

###  解释

**浏览器多次对Servlet的请求，** 一般情况下， **服务器只创建一个Servlet对象，** 也就是说，Servlet对象 **一旦创建了，**
就会 **驻留在内存中，为后续的请求做服务，直到服务器关闭。**

**每次访问请求对象和响应对象都是新的**

对于每次访问请求，Servlet引擎都会创建一个新的 **HttpServletRequest** 请求对象和一个新的
**HttpServletResponse** 响应对象，然后将这两个对象 **作为参数** 传递给它调用的Servlet的 ` service() `
方法， ` service() ` 方法再根据请求方式分别调用其他方法。

###  线程安全问题

当多个用户访问Servlet的时候， **服务器会为每个用户创建一个线程。** 当多个用户并发访问Servlet共享资源的时候就会出现线程安全问题。

###  原则

  1. 如果一个变量 **需要多个用户共享** ，则应当在访问该变量的时候，需要加锁 
  2. 如果一个变量 **不需要共享，** 则 **直接在 doGet() 或者 doPost()定义** ，这样不会存在线程安全问题 

* * *

#  通过注解配置 Servlet

在之前的开发工作中，每次编写一个Servlet都需要在 **web.xml** 文件中进行配置

    
    
    <servlet>
        <servlet-name>ActionServlet</servlet-name>
        <servlet-class>com.web.controller.ActionServlet</servlet-class>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>ActionServlet</servlet-name>
        <url-pattern>/servlet/ActionServlet</url-pattern>
    </servlet-mapping>
    

而当一个项目中存在很多 Servlet，那么配置文件就会变得很乱很大，在 Servlet 3.0 推出之后，我们可以使用注解来配置 Servlet

    
    
    @WebServlet(name = "ActionServlet", urlPatterns = "/servlet/ActionServlet")
    

* * *

#  Web 组件之间的跳转方式

##  请求转发（forward）

又叫做 **直接转发方式，** 客户端和浏览器 **只发出一次请求，** Servlet、HTML、JSP或其它信息资源，由
**第二个信息资源响应该请求，** 在请求对象request中，保存的对象对于 **每个信息资源是共享的。**

比如：从 AServlet 请求转发到 BServlet

[ ![img](https://upload-
images.jianshu.io/upload_images/7896890-881fc9bb05d46ac8.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1240) ](https://upload-
images.jianshu.io/upload_images/7896890-881fc9bb05d46ac8.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1240)

  * **语法：**

    
    
    request.getRequestDispatcher(path).forward(request, response);
    

_参数：_ ` path ` ，要跳转到的资源路径： **上下文路径 / 资源路径**

  * **特点：**
    * 地址栏中的地址不变 
    * 只有一个请求 
    * 资源是共享的 
    * 可以访问 WEB-INF 中的资源 
    * 请求转发 **不能** 跨域访问 

##  URl 重定向（redirect）

又叫做 **间接转发方式（Redirect）** 实际是 **两次HTTP请求，**
服务器端在响应第一次请求的时候，让浏览器再向另外一个URL发出请求，从而达到转发的目的。

比如:从AServlet重定向到BServlet

[ ![img](https://upload-
images.jianshu.io/upload_images/7896890-c49539085575bc26.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1240) ](https://upload-
images.jianshu.io/upload_images/7896890-c49539085575bc26.png?imageMogr2/auto-
orient/strip%7CimageView2/2/w/1240)

  * **语法：**

    
    
    response.sendRedirect(String location);
    

_参数：_ ` location ` ，转发到的资源路径

  * **特点：**
    * 地址栏中的地址【会】发生改变 
    * 有两个请求 
    * 在两个 Servlet 中不可以共享请求中的数据 
    * 最终的响应由BServlet来决定，和 AServlet 没有关系 
    * 不可以访问 WEB-INF 中的资源 
    * 请求转发 **可以** 跨域访问 

#  参考

  * [ https://how2j.cn/k/servlet/servlet-eclipse/558.html ](https://how2j.cn/k/servlet/servlet-eclipse/558.html)

  * [ https://www.yiibai.com/servlet ](https://www.yiibai.com/servlet)

  * [ https://www.runoob.com/servlet/servlet-tutorial.html ](https://www.runoob.com/servlet/servlet-tutorial.html)

  * [ https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483680&idx=3&sn=d5380ff58c5077271ac9c43d2d96f6c1&chksm=ebd74021dca0c93733255324df8c1e522dbe36ccaf8c2c4bcca4765113a120eb9851ca0e2442#rd ](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483680&idx=3&sn=d5380ff58c5077271ac9c43d2d96f6c1&chksm=ebd74021dca0c93733255324df8c1e522dbe36ccaf8c2c4bcca4765113a120eb9851ca0e2442#rd)

  * [ https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483680&idx=4&sn=2fdf4d0075d093389c03697ebdb9f47d&chksm=ebd74021dca0c937a240f47578b9c5f40093a307f6537d79d5a2fd12721c5311a9d89d5c5583#rd ](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483680&idx=4&sn=2fdf4d0075d093389c03697ebdb9f47d&chksm=ebd74021dca0c937a240f47578b9c5f40093a307f6537d79d5a2fd12721c5311a9d89d5c5583#rd)

  * [ https://www.jianshu.com/p/bbdc459b9187 ](https://www.jianshu.com/p/bbdc459b9187)

