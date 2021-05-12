---
title: 轻松理解 Java 静态代理/动态代理
date: 2021-04-12 10:25
categories: ['Java']
tags: ['Java']
---
#  什么是代理模式

在有些情况下，一个客户不能或者不想直接访问另一个对象，这时需要找一个中介帮忙完成某项任务，这个中介就是代理对象。

例如，购买火车票不一定要去火车站买，可以通过 12306 网站或者去火车票代售点买。又如找女朋友、找保姆、找工作等都可以通过找中介完成。

##  定义

由于某些原因需要给某对象提供一个代理以控制对该对象的访问。

访问对象 **不适合或者不能直接引用** 目标对象， **代理对象** 作为 **访问对象** 和 **目标对象** 之间的 **中介** 。

##  代理模式的主要角色

  * 抽象角色（Subject）：通过接口或抽象类声明真实主题和代理对象实现的业务方法。 

  * 真实角色（Real Subject）：实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象。 

  * 代理（Proxy）：提供了与真实主题相同的接口，其内部含有对真实主题的引用，它可以访问、控制或扩展真实主题的功能。 

  * 客户 : 使用代理角色来进行一些操作 . 

##  优点

  * 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用 
  * 代理对象可以扩展目标对象的功能 
  * 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度，增加了程序的可扩展性 

##  缺点

  * 冗余，由于代理对象要实现与目标对象一致的接口，会产生过多的代理类。 
  * 系统设计中类的数量增加，变得难以维护。 

> 使用动态代理方式，可以有效避免以上的缺点

#  静态代理

静态代理其实就是最基础、最标准的代理模式实现方案。

**举例：**

Rent . java 即抽象角色

    
    
    //抽象角色：租房
    public interface Rent {
       public void rent();
    }
    

Landlord . java 即真实角色

    
    
    //真实角色: 房东，房东要出租房子
    public class Landlord implements Rent{
       public void rent() {
           System.out.println("房屋出租");
      }
    }
    

Proxy . java 即代理

    
    
    //代理角色：中介
    public class Proxy implements Rent {
    
       private Landlord landlord;
       public Proxy() { }
       public Proxy(Landlord landlord) {
           this.landlord = landlord;
      }
       //租房
       public void rent(){
           seeHouse();
           landlord.rent();
           fare();
      }
       //看房
       public void seeHouse(){
           System.out.println("带房客看房");
      }
       //收中介费
       public void fare(){
           System.out.println("收中介费");
      }
    }
    

Client . java 即客户

    
    
    //客户类，一般客户都会去找代理！
    public class Client {
       public static void main(String[] args) {
           //房东要租房
           Landlord landlord = new Landlord();
           //中介帮助房东
           Proxy proxy = new Proxy(landlord);
           //客户找中介
           proxy.rent();
      }
    }
    

**结果：**

    
    
    带房客看房
    房屋出租
    收中介费
    
    Process finished with exit code 0
    

在这个过程中，客户接触的是中介，看不到房东，但是依旧租到了房东的房子。同时房东省了心，客户省了事。

静态代理享受代理模式的优点，同时也具有代理模式的缺点，那就是一旦实现的功能增加，将会变得异常冗余和复杂，秒变光头。

为了保护头发，就出现了动态代理模式！

#  动态代理

动态代理的出现就是为了解决传统静态代理模式的中的缺点。

具备代理模式的优点的同时，巧妙的解决了静态代理代码冗余，难以维护的缺点。

在Java中常用的有如下几种方式：

  * JDK 原生动态代理 
  * cglib 动态代理 
  * javasist 动态代理 

#  JDK原生动态代理

上例中静态代理类中，中介作为房东的代理，实现了相同的租房接口。

##  例子

  1. 首先实现一个InvocationHandler，方法调用会被转发到该类的invoke()方法。 
  2. 然后在需要使用Rent的时候，通过JDK动态代理获取Rent的代理对象。 

    
    
    class RentInvocationHandler implements InvocationHandler {
    
        private Rent rent;
    
        public RentInvocationHandler(Rent rent) {
            this.rent = rent;
        }
    
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            seeHouse();
            Object result = method.invoke(rent, args);
            fare();
            return result;
        }
    
        //看房
        public void seeHouse(){
            System.out.println("带房客看房");
        }
        //收中介费
        public void fare(){
            System.out.println("收中介费");
        }
        //动态获取代理
        public Object getProxy() {
            return Proxy.newProxyInstance(this.getClass().getClassLoader(),
                    rent.getClass().getInterfaces(),this); //核心关键
        }
    }
    

客户使用动态代理调用

    
    
    public class Client {
        public static void main(String[] args) {
            Landlord landlord = new Landlord();
            //代理实例的调用处理程序
            RentInvocationHandler pih = new RentInvocationHandler(landlord);
            Rent proxy = (Rent)pih.getProxy(); //动态生成对应的代理类！
            proxy.rent();
        }
    }
    

运行结果和前例相同

##  分析

上述代码的核心关键是 ` Proxy.newProxyInstance ` 方法，该方法会根据指定的参数动态创建代理对象。

它三个参数的意义如下：

  1. ` loader ` ，指定代理对象的类加载器 
  2. ` interfaces ` ，代理对象需要实现的接口，可以同时指定多个接口 
  3. ` handler ` ，方法调用的实际处理者，代理对象的方法调用都会转发到这里 

` Proxy.newProxyInstance ` 会返回一个实现了指定接口的代理对象，对该对象的所有方法调用都会转发给 `
InvocationHandler.invoke() ` 方法。

因此，在 ` invoke() ` 方法里我们可以加入任何逻辑，比如修改方法参数，加入日志功能、安全检查功能等等等等……

##  小结

显而易见，对于静态代理而言，我们需要 **手动编写代码** 让 **代理** 实现 **抽象角色** 的接口。

而在动态代理中，我们可以让程序在运行的时候 **自动在内存中创建** 一个实现 **抽象角色接口**
的代理，而不需要去单独定义这个类，代理对象是在程序运行时产生的，而不是编译期。

> 对于从Object中继承的方法，JDK Proxy会把 ` hashCode() ` 、 ` equals() ` 、 ` toString() `
> 这三个非接口方法转发给 ` InvocationHandler ` ，其余的Object方法则不会转发。

#  CGLIB动态代理

JDK动态代理是基于接口的，如果对象没有实现接口该如何代理呢？CGLIB代理登场

> [ CGLIB ](https://github.com/cglib/cglib) ( _Code Generation Library_ )是一个基于
> [ ASM ](http://www.baeldung.com/java-asm)
> 的字节码生成库，它允许我们在运行时对字节码进行修改和动态生成。CGLIB通过继承方式实现代理。

使用cglib需要引入cglib的jar包，如果你已经有spring-core的jar包，则无需引入，因为spring中包含了cglib。

    
    
    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>3.3.0</version>
    </dependency>
    

##  例子

来看示例，假设我们有一个没有实现任何接口的类 ` Landlord ` ：

    
    
    public class Landlord{
        public void rent() {
            System.out.println("房屋出租");
        }
    }
    

因为没有实现接口，所以使用通过CGLIB代理实现如下：

首先实现一个MethodInterceptor，方法调用会被转发到该类的intercept()方法

    
    
    public class RentMethodInterceptor implements MethodInterceptor {
        private Object target;//维护一个目标对象
        public RentMethodInterceptor(Object target) {
            this.target = target;
        }
        //为目标对象生成代理对象
        public Object getProxyInstance() {
            //工具类
            Enhancer en = new Enhancer();
            //设置父类
            en.setSuperclass(target.getClass());
            //设置回调函数
            en.setCallback(this);
            //创建子类对象代理
            return en.create();
        }
        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            System.out.println("看房");
            // 执行目标对象的方法
            Object returnValue = method.invoke(target, objects);
            System.out.println("中介费");
            return null;
        }
    }
    

客户通过CGLIB动态代理获取代理对象

    
    
    public class Client {
        public static void main(String[] args) {
            Landlord target = new Landlord();
            System.out.println(target.getClass());
            //代理对象
            Landlord proxy = (Landlord) new RentMethodInterceptor(target).getProxyInstance();
            System.out.println(proxy.getClass());
            //执行代理对象方法
            proxy.rent();
        }
    }
    

运行输出结果和前例相同

##  分析

> 对于从Object中继承的方法，CGLIB代理也会进行代理，如 ` hashCode() ` 、 ` equals() ` 、 ` toString()
> ` 等，但是 ` getClass() ` 、 ` wait() ` 等方法不会，因为它是final方法，CGLIB无法代理。

其实CGLIB和JDK代理的思路大致相同

上述代码中，通过CGLIB的 ` Enhancer ` 来指定要代理的目标对象、实际处理代理逻辑的对象。

最终通过调用 ` create() ` 方法得到代理对象， **对这个对象所有非final方法的调用都会转发给`
MethodInterceptor.intercept() ` 方法 ** 。

在 ` intercept() ` 方法里我们可以加入任何逻辑，同JDK代理中的 ` invoke() ` 方法

通过调用 ` MethodProxy.invokeSuper() ` 方法，我们将调用转发给原始对象，具体到本例，就是 ` Landlord `
的具体方法。CGLIG中 ` MethodInterceptor ` 的作用跟JDK代理中的 ` InvocationHandler `
很类似，都是方法调用的中转站。

##  final类型

CGLIB是通过继承的方式来实现动态代理的，有继承就不得不考虑final的问题。我们知道final类型不能有子类，所以CGLIB不能代理final类型，遇到这种情况会抛出类似如下异常：

    
    
    java.lang.IllegalArgumentException: Cannot subclass final class cglib.HelloConcrete
    

同样的，final方法是不能重载的，所以也不能通过CGLIB代理，遇到这种情况不会抛异常，而是会跳过final方法只代理其他方法。

##  其他方案

  * 使用ASM在被代理类基础上生成新的字节码形成代理类 
  * 使用javassist在被代理类基础上生成新的字节码形成代理类 

javassist也是常用的一种动态代理方案，ASM速度非常快，这里不在进行展开。

#  尾声

动态代理是 [ Spring AOP ](https://docs.spring.io/spring/docs/current/spring-
framework-reference/core.html#aop) ( _Aspect Orient Programming, 面向切面编程_
)的实现方式，了解动态代理原理，对理解Spring AOP大有帮助。

  * 如spring等这样的框架，要增强具体业务的逻辑方法，不可能在框架里面去写一个静态代理类，太蠢了，只能按照用户的注解或者xml配置来动态生成代理类。 
  * 业务代码内，当需要增强的业务逻辑非常通用（如:添加log，重试，统一权限判断等）时，使用动态代理将会非常简单，如果每个方法增强逻辑不同，那么静态代理更加适合。 
  * 使用静态代理时，如果代理类和被代理类同时实现了一个接口，当接口方法有变动时，代理类也必须同时修改，代码将变得臃肿且难以维护。 

