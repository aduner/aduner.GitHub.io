---
title: 轻松理解 Spring AOP 
date: 2021-04-14 09:40
categories: ['Spring', 'Java']
tags: ['SpringBoot', 'Spring', 'Java']
---
#  Spring AOP 简介

##  Spring AOP 的基本概念

AOP (Aspect-Oriented Programming)，即 **面向切面编程** , 它与 OOP (Object-Oriented
Programming, 面向对象编程) 相辅相成, 提供了与 OOP 不同的抽象软件结构的视角.

在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 **Aspect(切面)**

AOP是 Spring 是最难理解的概念之一，同时也是非常重要的知识点，因为它真的很常用。

###  面向切面编程

在面向切面编程的思想里面，把功能分为两种

  * **核心业务** ：登陆、注册、增、删、改、查、都叫核心业务 
  * **周边功能** ：日志、事务管理这些次要的为周边业务 

在面向切面编程中，核心业务功能和周边功能是分别独立进行开发，两者不是耦合的；

然后把切面功能和核心业务功能 "编织" 在一起，这就叫AOP

##  AOP 的目的

AOP能够将那些与业务无关， **却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来** ，便于 **减少系统的重复代码**
， **降低模块间的耦合度** ，并 **有利于未来的可拓展性和可维护性** 。

#  AOP 术语和流程

进一步了解AOP之前，我们先来看看AOP中使用到的一些术语，以及AOP运行的流程。

##  术语

  * **连接点（join point）** ：对应的是具体被拦截的对象，因为Spring只能支持方法，所以被拦截的对象往往就是指特定的方法。 **具体是指一个方法**
  * **切点（point cut）** ：有时候，我们的切面不单单应用于单个方法，也可能是 **多个类的不同方法** ，这时，可以通过正则式和指示器的规则去定义，从而适配连接点。切点就是提供这样一个功能的概念。 **具体是指具体共同特征的多个方法。**
  * **通知（advice）** ：它会根据约定织入流程中，需要弄明白它们在流程中的顺序和运行的条件，有这几种： 
    * 前置通知（before advice） 
    * 环绕通知（around advice） 
    * 后置通知（after advice） 
    * 异常通知（afterThrowing advice） 
    * 事后返回通知（afterReturning advice） 
  * **目标对象（target）：** 即被代理的对象，通俗理解各个切点的所在的类就是目标对象。 
  * **引入（introduction）** ：是指引入新的类和其方法，增强现有Bean的功能。 
  * **织入（weaving）** ：它是一个通过 **动态代理技术** ，为原有服务对象生成代理对象，然后将与切点定义匹配的连接点拦截，并按约定将各类通知织入约定流程的过程。 
  * **切面（aspect）** ：定义切点、各类通知和引入的内容，AOP将通过它的信息来增强Bean的功能或将对应的方法织入流程。 

上述的描述还是比较抽象的，配合下面的流程讲解以及例子，应该充分掌握这些概念了。

##  流程

画了一张图，通过张图可以清晰的了解AOP的整个流程，以及上面各个术语的意义和关系。

> 图片的流程顺序基于Spring 5

![image-20210414091230633](https://tva1.sinaimg.cn/large/008eGmZEly1gpj0655wl1j31bf0u0gqf.jpg)

###  五大通知执行顺序

不同版本的Spring是有一定差异的，使用时候要注意

  * Spring 4 

    * 正常情况：环绕前置 **== > ** @Before **== > ** 目标方法执行 **== > ** 环绕返回 **== > ** 环绕最终 **== > ** @After **== > ** @AfterReturning 
    * 异常情况：环绕前置 **== > ** @Before **== > ** 目标方法执行 **== > ** 环绕异常 **== > ** 环绕最终 **== > ** @After **== > ** @AfterThrowing 
  * Spring 5.28 

    * 正常情况：环绕前置 **== > ** @Before **== > ** 目标方法执行 **== > ** @AfterReturning **== > ** @After **== > ** 环绕返回 **== > ** 环绕最终 
    * 异常情况：环绕前置 **== > ** @Before **== > ** 目标方法执行 **== > ** @AfterThrowing **== > ** @After **== > ** 环绕异常 **== > ** 环绕最终 

##  例子

###  图例

举一个实际中的例子来说明一下方便理解：

![image-20210413110355452](https://tva1.sinaimg.cn/large/008eGmZEly1gphxrp5milj30tw0m2abz.jpg)

房东的核心诉求其实就是签合同，收钱，浅绿部分都是次要的，交给中介就好。

不过有的人可能就有疑问了，让房东带着不是更好吗，租客沟通起来不是更轻松吗？为啥非要分成两部分呢？

**那么请看下面这种情况**

![image-20210413111018593](https://tva1.sinaimg.cn/large/008eGmZEly1gphxyk0pz2j310q0u0gqi.jpg)

当我们有很多个房东的时候，中介的优势就体现出来了。代入到我们实际的业务中，AOP能够极大的减轻我们的开发工作， **让关注点代码与业务代码分离！**
实现解藕！

###  实际的代码

用一个实际代码案例来感受一下

  1. 创建一个房东 

    
    
    @Component("landlord")
    public class Landlord {
    
    	public void service() {
    		System.out.println("签合同");
    		System.out.println("收钱");
    	}
    }
    

  2. 创建中介 

    
    
    @Component
    @Aspect
    class Broker {
    	@Before("execution(* pojo.Landlord.service())")
    	public void before(){
    		System.out.println("带租客看房");
    		System.out.println("谈钱");
    	}
    	@After("execution(* pojo.Landlord.service())")
    	public void after(){
    		System.out.println("给钥匙");
    	}
    }
    

3.在 applicationContext.xml 中配置自动注入

    
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
        <context:component-scan base-package="aspect" />
        <context:component-scan base-package="pojo" />
        <aop:aspectj-autoproxy/>
    </beans>
    

  4. 测试 

    
    
    public class Test {
    	public static void main(String[] args) {
    		ApplicationContext context =
    				new ClassPathXmlApplicationContext("applicationContext.xml");
    		Landlord landlord = (Landlord) context.getBean("landlord", Landlord.class);
    		landlord.service();
    	}
    }
    

5.执行看到效果：

    
    
    带租客看房
    谈钱
    签合同
    收钱
    给钥匙
    

这个例子中我们用到了 ` @Before ` 和 ` @After ` 两个注解，其实就是设置的前置通知和后置通知。

最后的结果似乎与我们之前图例中的顺序不同，给钥匙在收钱之后了，这个问题留到后面再解决，目前只需要简单感受一下aop的使用即可。

> 预告：这种情况下应该使用 **环绕通知** 来完成这个需求

* * *

#  使用 Spring AOP

##  使用注解开发AOP

目前使用注解的方式进行Spring开发才是主流，包括SpringBoot中，已经是全注解开发，所以我们采用@AspectJ的注解方式，重新实现一下上面的用例，来学习AOP的使用。

注解  |  说明  
---|---  
` @Before ` |  前置通知，在连接点方法前调用  
` @Around ` |  环绕通知，它将覆盖原有方法，可以想象成前置+原方法+后置  
` @After ` |  后置通知，在连接点方法后调用  
` @AfterReturning ` |  返回通知，在连接点方法执行并正常返回后调用，要求连接点方法在执行过程中没有发生异常  
` @AfterThrowing ` |  异常通知，当连接点方法异常时调用  
  
###  第一步：选择连接点

Spring 是方法级别的 AOP 框架，我们主要也是以某个类额某个方法作为连接点，另一种说法就是： **选择哪一个类的哪一方法用以增强功能。**

    
    
    @Component
    public class Landlord {
    	public void service() {
    		System.out.println("签合同");
    		System.out.println("收钱");
    	}
    }
    

我们在这里就选择上述 Landlord 类中的 service() 方法作为连接点。

###  第二步：创建切面

选择好了连接点就可以创建切面了，我们可以把切面理解为一个拦截器，当程序运行到连接点的时候，被拦截下来，在开头加入了初始化的方法，在结尾也加入了销毁的方法而已，在
Spring 中只要使用 ` @Aspect ` 注解一个类，那么 Spring IoC 容器就会认为这是一个切面了：

    
    
    @Component
    @Aspect
    class Broker {
    
        @Before("execution(* com.aduner.demo03.pojo.Landlord.service())")
        public void before(){
            System.out.println("带租客看房");
            System.out.println("谈钱");
        }
    
        @After("execution(* com.aduner.demo03.pojo.Landlord.service())")
        public void after(){
            System.out.println("给钥匙");
        }
    }
    

> 切面的类仍然是一个 Bean ，需要 ` @Component ` 注解标注

在上面的注解中定义了 execution 的正则表达式，Spring会通过这个正则式去匹配、去确定对应的方法（连接点）是否启用切面编程

    
    
    execution(* com.aduner.demo03.pojo.Landlord.service())
    

依次对这个表达式作出分析：

  * execution：执行方法的时候会触发 
  * ` * ` ：任意返回类型的方法 
  * com.aduner.demo03.pojo.Landlord：类的全限定名 
  * service()：拦截的方法的名称 
  * 如果是service(*) 就是表示任意参数的service方法 

###  第三步：定义切点

每一个注解都重复写了同一个正则式，这显然比较冗余。为了克服这个问题，Spring定义了切点（Pointcut）的概念，切点的作用就是向Spring描述哪些类的哪些方法需要启用AOP编程，这样可以有效的降低代码的复杂度，而且有利于维护的方便。

    
    
    @Component
    @Aspect
    class Broker {
    
        @Pointcut("execution(* com.aduner.demo03.pojo.Landlord.service())")
        public void pointcut() {
        }
    
        @Before("pointcut()")
        public void before() {
            System.out.println("带租客看房");
            System.out.println("谈钱");
        }
    
        @After("pointcut()")
        public void after() {
            System.out.println("给钥匙");
        }
    }
    

###  第四步：配置好config

    
    
    @Configuration
    @EnableAspectJAutoProxy
    @ComponentScan(basePackages = "com.aduner.demo03.*",
            excludeFilters = {@ComponentScan.Filter(classes = {Service.class})})
    public class AppConfig {
    
    }
    

###  第五步：测试 AOP

    
    
    @Test
    void testAspect(){
        ApplicationContext ctx = new AnnotationConfigApplicationContext( AppConfig.class ) ;
        Landlord landlord=ctx.getBean(Landlord.class);
        landlord.service();
        ((ConfigurableApplicationContext)ctx).close();
    }
    

结果

    
    
    ……
    带租客看房
    谈钱
    签合同
    收钱
    给钥匙
    ……
    

##  环绕通知

现在我们来解决一下前面遗留的那个问题，收钱和给钥匙的问题。

我们需要的应该是给了钥匙之后再收钱，但是现在是反过来的。

要实现这个需求，用到环绕通知，这是 Spring AOP 中最强大的通知，集成了前置通知和后置通知。

**环绕通知（Around）**
是所有通知中最为强大的通知，强大也意味着难以控制。一般而言，使用它的场景是在你需要大幅度修改原有目标对象的服务逻辑时，否则都尽量使用其他的通知。

环绕通知是一个取代原有目标对象方法的通知，当然它也提供了回调原有目标对象方法的能力。

  1. 我们先来修改一下Landlord 

    
    
    @Component
    public class Landlord {
        public void service(int steps) {
            if (steps == 1) {
                System.out.println("签合同");
            } else if(steps==2){
                System.out.println("收钱");
            }
            else {
                System.out.println("签合同");
                System.out.println("收钱");
            }
    
        }
    }
    

我们将service添加一个参数，第一步签合同，第二部收钱，如果没有制定第一步或者第二步，就一起执行。

  2. 然后重新编写一下我们的切面 

    
    
    @Component
    @Aspect
    class Broker {
        @Pointcut("execution(* com.aduner.demo03.pojo.Landlord.service(*))")
        public void pointcut() {
        }
      
        @Around("pointcut()")
        public void around(ProceedingJoinPoint joinPoint) {
            System.out.println("带租客看房");
            System.out.println("谈价格");
    
            try {
              	// joinPoint.proceed(); 这样就是执行原方法
                joinPoint.proceed(new Object[]{1}); //重新指定方法的参数
                System.out.println("交钥匙");
                joinPoint.proceed(new Object[]{2});
            } catch (Throwable throwable) {
                throwable.printStackTrace();
            }
        }
    }
    

  3. 修改一下刚刚的测试类，给到一个初始参数 

    
    
    @Test
    void testAspect(){
      ApplicationContext ctx = new AnnotationConfigApplicationContext( AppConfig.class ) ;
      Landlord landlord=ctx.getBean(Landlord.class);
      landlord.service(0);
      ((ConfigurableApplicationContext)ctx).close();
    }
    

运行！成功！

    
    
    ……
    带租客看房
    谈价格
    签合同
    交钥匙
    收钱
    ……
    

####  ProceedingJoinPoint对象

注意到切面编写中Around里面 ` try ` 中的 ` joinPoint.proceed() ` 方法

ProceedingJoinPoint对象是JoinPoint的子接口，该对象只用在@Around的切面方法中，添加了以下两个方法。

    
    
    Object proceed() throws Throwable //执行目标方法 
    Object proceed(Object[] var1) throws Throwable //传入的新的参数去执行目标方法 
    

前面的例子中我们显示把房东的工作分为了两步，然后再环绕通知中重新赋予参数并调用了两次，在两次中间插入了中介的工作。

实际开发中，上面这样的写法其实又会造成新的耦合，而且 **还会造成其他通知的混乱** （调用了两次方法，其实会让有些通知返回两次）。

当然 **这只是一个例子** ，为了帮助更好的理解环绕通知。

##  多个切面

Spring可以支持多个切面同时运行，如果刚好多个切面的切点相同，切面的运行顺序就是一个关键了。

默认情况下，切面的运行顺序是混乱的，如果需要指定切面的运行顺序，我们需要用到 ` @Order ` 注解

    
    
    @Component
    @Aspect
    @Order(1)
    public class FirstAspect {
      ……
    }
    --------------------
    @Component
    @Aspect
    @Order(2)
    public class SecondAspect {
      ……
    }
    

` @Order ` 注解中的值就是切片的顺序，但是他们不是顺序执行的而是包含关系。

![image-20210414092656057](https://tva1.sinaimg.cn/large/008eGmZEly1gpj0lc64sjj30q20g6dgv.jpg)

#  总结

  * AOP的出现是为了对程序解耦， **减少系统的重复代码，提高可拓展性和可维护性** 。 
  * 常见的应用场景有权限管理、缓存、记录跟踪、优化、校准、日志、事务等等等等……总之AOP的使用是非常常见的。 
  * 需要注意不同Spring版本之间的 **AOP通知顺序是有差别** 的。补充：Spring5.28为分界线。 
  * 环绕通知很灵活、强大，但是也就意味着很难控制， **如非必要** ，优先使用其他通知来完成。 
  * 多切面作用同一个切点时候注意切片顺序。 

