---
title: 轻松理解 Spirng IoC/控制反转
date: 2021-04-11 18:38
categories: ['Java', 'Spring']
tags: ['后端', 'Java', 'Spring', 'SpringBoot']
---
#  Spring IoC 概述

##  IoC：Inverse of Control（控制反转）

> 控制反转不是一种技术，而是一种 **思想** 。

既然说是 **反** 转就说先明白什么是 **正** ，什么是 **反**

  * **正控：** 就是我们平时最常见的那种使用形式，要使用某个对象，需要 **自己去负责对象的创建** ，属于自力更生。 
  * **反控：** 若要使用某个对象，无需自己创建，只需要 **从IoC容器中去获取** ，创建对象的过程交给Spring来处理，Spring来维护这个IoC容器，属于富二代，需要啥找管家。 

**所谓控制反转，就是把原先我们代码里面需要实现的对象创建、依赖的代码，通过描述反转给容器来帮忙实现。**

**在 Java 中可以是 XML 或者注解，通过Spring去产生或获取特定对象**

Spring 会提供 **IoC 容器** 来管理和容纳我们所开发的各种各样的 Bean，并且我们可以从中获取各种发布在 Spring IoC 容器里的
Bean，并且 **通过描述** 可以得到它。

###  一个例子

比如我想要一个机器人，我有两种方法：

  * 造出来 
  * 买过来 

![image-20210411164802090](https://tva1.sinaimg.cn/large/008eGmZEly1gpfwh7m8s9j31ay0rsdjj.jpg)

正控的方式其实就是需要机器人的时候自己造，显然它不如买的方便，开销也不见得比买的小，造出来的未必比买的好。

IoC其实就相当于是买，这个过程中我们没有去 **创造** 机器人，但是最终也得到了机器人，而且大概率要比我们自己造的好。

最终的结果都是得到机器人，关键的区别就在于，机器人是谁创造的。如果我们还需要其他的东西例如、汽车、电视等等，自己就造就显得很蠢，找人买显然更聪明。

##  Spring IoC 的好处

当上面的例子作用于庞杂的软件工程中的时候，自己造的方式显然是难以维护的。

好处显而易见：

  * 降低对象之间的耦合 
  * 不需要理解一个类的具体实现，直接向 IoC 容器拿 

#  IoC实例

>
> ClassPathXmlApplicationContext是ApplicationContext的子类，ApplicationContext下面有所介绍

我们先来看一个最简单例子

  1. 创建一个实体类 

    
    
    public class Source {
    	private String taste;
    	private String sugar;
      private String size;
      
      /**setter and getter **/
    }
    

  2. 先在 ` src ` 目录下创建一个 ` bean.xml ` 文件： 

    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        <!-- 通过 xml 方式装配 bean -->
        <bean name="source" class="pojo.Source">
            <property name="taste" value="苹果味"/>
            <property name="sugar" value="糖"/>
            <property name="size" value="中杯"/>
        </bean>
    </beans>
    

  3. 上面定义了一个 bean ，这样 Spring IoC 容器在初始化的时候就能找到它们，然后使用 ClassPathXmlApplicationContext 容器就可以将其初始化： 

    
    
    ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
    Source source = (Source) context.getBean("source", Source.class);
    
    System.out.println(source.getTaste());
    System.out.println(source.getSugar());
    System.out.println(source.getSize());
    

这样就会使用 Application 的实现类 ClassPathXmlApplicationContext 去初始化 Spring IoC
容器，然后开发者就可以通过 IoC 容器来获取资源了。

> bean的装配还可以通过注解的方式，SpringBoot中常用注解来装配，文末有案例

#  Spring IoC 容器的设计

##  设计

Spring Bean的创建是典型的工厂模式，这一系列的Bean工厂，也即IOC容器为开发者管理对象间的依赖关系提供了很多便利和基础服务。

Spring提供了多个IoC容器可供使用。

Spring IoC 容器的设计主要是基于以下两个接口：

  * **BeanFactory** （Spring IoC 容器所定义的最底层接口） 
  * **ApplicationContext**

ApplicationContext 是 BeanFactory 的子接口之一，并对 BeanFactory
功能做了许多的扩展大多数时候，都是使用它来作为IoC容器

![img](https://tva1.sinaimg.cn/large/008eGmZEly1gpfx2wgkptj30h40gagn3.jpg)

##  BeanFactory

  * 是Spring中最底层的接口，只提供了最简单的IoC功能，负责配置，创建和管理bean。 
  * 在应用中，一般不使用 BeanFactory，而推荐使用ApplicationContext（应用上下文）。 

从上图可以看到， BeanFactory 位于设计的最底层，它提供了 Spring IoC 最底层的设计，下面简单说下它所定义的方法

  * ` getBean() ` 对应了多个方法来获取配置给 Spring IoC 容器的 Bean。 
    * 按照 **类型** 拿 bean（有多个 **type** Spring 就懵了，不知道该获取哪一个） 
    * 按照 bean 的 **名字** 拿 bean 
    * 按照名字和类型拿 bean **（推荐）**

> **单例** 就是无论获取多少次，都是同一个对象。 **原型** 就是每次获取都是一个新创建的对象。一般 **默认是单例的**

  * ` isSingleton() ` 用于判断是否单例 
  * ` isPrototype() ` 用于判断是否为原型 
  * ` isTypeMatch() ` 是否匹配类型 
  * ` getType() ` 获取bean的类型 
  * ` getAliases() ` 方法是获取别名的方法 
  * ` containsBean() ` 是否包含bean 

所有关于 Spring IoC 的容器将会遵守 ` BeanFactory ` 所定义的方法。

##  ApplicationContext

根据 ApplicationContext 的类继承关系图，可以看到 ApplicationContext 接口扩展了许许多多的接口，因此它的功能十分强大。

  * 继承了 BeanFactory，拥有了基本的 IoC 功能； 
  * 支持国际化； 
  * 支持消息机制； 
  * 支持统一的资源加载； 
  * 支持AOP功能； 

####  ApplicationContext 常见实现类：

1\. **ClassPathXmlApplicationContext**

读取classpath中的资源

    
    
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    

2: **FileSystemXmlApplicationContext**

读取指定路径的资源

    
    
    ApplicationContext ac = new FileSystemXmlApplicationContext("/projact/applicationContext.xml");
    

3\. **XmlWebApplicationContext**  
需要在Web的环境下才可以运行

    
    
    XmlWebApplicationContext ac = new XmlWebApplicationContext(); // 这时并没有初始化容器
    ac.setServletContext(servletContext); // 需要指定ServletContext对象
    ac.setConfigLocation("/WEB-INF/applicationContext.xml"); // 指定配置文件路径，开头的斜线表示Web应用的根目录
    ac.refresh(); // 初始化容器
    

* * *

#  Bean的定义与初始化

Bean 的定义和初始化是两个步骤

**定义：**

  1. Resource 定位   
Spring IoC 容器先根据开发者的配置，进行资源的定位，通过 **XML 或者注解** ，定位的内容是由开发者提供的。

  2. BeanDefinition 的载入   
这个时候只是将 Resource 定位到的信息，保存到 Bean 定义（BeanDefinition）中，此时并不会创建 Bean 的实例

  3. BeanDefinition 的注册 

这个过程就是将 BeanDefinition 的信息发布到 Spring IoC 容器中，此时仍然没有对应的 Bean 的实例。

做完了以上 3 步，Bean 就在 Spring IoC 容器中被定义了，但是没有被初始化、没有完成依赖注入，它还不能使用。

**初始化和依赖注入：**

Bean 有一个配置选项—— **` lazy-init ` ** ，作用在于是否懒加载。

> 懒加载的意思就是，如果不获取它，就不创建实例，当获取它时才创建实例。通俗理解就是，不到饭点不起床。

在没有任何配置的情况下，它的默认值为 false，也就是 **默认会自动初始化 Bean** 。

如果将其设置为 true，那么只有当我们使用 Spring IoC 容器的 ` getBean() ` 方法获取它时，它才会进行 Bean
的初始化，完成依赖注入。

* * *

#  依赖注入(DI)

**依赖注入** 就是将实例变量传入到一个对象中去(Dependency injection means giving an object its
instance variables)。

##  什么是依赖

例如

    
    
    public class Human {
        ...
        Father father;
        ...
        public Human() {
            father = new Father();
        }
    }
    

上面这段代码中，Human就是依赖于Father，如果Father出错，Human类就无法正常运行。

##  什么是依赖注入

上面将依赖在构造函数中直接初始化是一种硬编码，弊端在于两个类不够独立。

    
    
    public class Human {
        ...
        Father father;
        ...
        public Human(Father father) {
            this.father = father;
        }
    }
    

我们将 father 对象作为构造函数的一个参数传入，在调用 Human 的构造方法之前外部就已经初始化好了 Father 对象。

这种非自己主动初始化依赖，而通过外部来传入依赖的方式，我们就称为依赖注入。

这种方式的好处显而易见，那就是降低了耦合。

##  IoC和DI的关系

有些人会把控制反转和依赖注入等同，但实际上它们有着本质上的不同。

  * **控制反转** 是一种思想 
  * **依赖注入** 是一种设计模式 

**IoC框架使用依赖注入作为实现控制反转的方式** ，Spring中的IoC就是使用DI的方式来实现的

但控制反转不止这一种实现方式，只是这种应用的更为广泛。

##  如何自己实现一个的IoC容器

简单聊一下，大概就分成三步：

  1. 读取注解或者配置文件，查看依赖，拿到类名 
  2. 使用反射，基于类名实例化对应的对象实例 
  3. 将对象实例，通过构造函数或者 setter，传递出去 

Spring的IoC大致就是这样一个路数，这其中还有更多的细节，但大致的思路就是如此。

#  SpringBoot中IoC的使用案例

在Spring Boot当中我们主要是通过注解来装配Bean到Spring IoC容器中。

  1. 首先定义一个Java简单对象 

    
    
    public class User {
        private Long id;
        private String userName;
        private String note;
    
        /**setter and getter **/
    }
    

  2. 再定义一个Java配置文件 
    * @Configuration代表这是一个Java配置文件，Spring的容器会根据它来生成IoC容器去装配Bean； 
    * @Bean代表将 ` initUser() ` 方法返回的POJO装配到IoC容器中，而其属性 ` name ` 定义这个Bean的名称，如果没有配置它，则将方法名称“ ` initUser ` ”作为Bean的名称保存到Spring IoC容器中。 

    
    
    @Configuration
    public class AppConfig {
        @Bean(name = "user")
        public User initUser() {
            User user = new User();
            user.setId(1L);
            user.setUserName("user_name_1");
            user.setNote("note_1");
            return user;
        }
    }
    

  3. 调用测试 

    
    
    public class IoCTest {
        private static Logger log = Logger.getLogger(IoCTest.class);
        public static void main(String[] args) {
            ApplicationContext ctx 
                 = new AnnotationConfigApplicationContext(AppConfig.class);
            User user = ctx.getBean(User.class);
            log.info(user.getId());
        }
    }
    
    
    
    ......
    17:53:03.018 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Returning cached instance of singleton bean 'user'
    17:53:03.018 [main] INFO com.springboot.chapter3.config.IoCTest - 1
    

显然，配置在配置文件中的名称为 ` user ` 的Bean已经被装配到IoC容器中，并且可以通过 ` getBean() `
方法获取对应的Bean，并将Bean的属性信息输出出来。

当然这只是很简单的方法，而注解@Bean也不是唯一创建Bean的方法，还有其他的方法可以让IoC容器装配Bean

#  总结

IoC不是什么技术，而是一种设计思想。在 ` Spring ` 开发中，由 ` IOC `
容器控制对象的创建、初始化、销毁等。这也就实现了对象控制权的反转，由 **我们对对象的控制** 转变成了 **` Spring IOC ` 对对象的控制
** 。

