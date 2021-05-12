---
title: Java并发/多线程-线程池的使用
date: 2021-01-17 19:27
categories: ['Java']
tags: ['多线程', 'Java']
---
#  线程池的优点

  * 线程频繁的 ` 创建=>销毁=>创建 ` 对系统对开销很大，使用线程池可以避免重复的开销 
  * 方便复用，提高相应速度 
  * 线程的创建于执行完全分开，方便维护，降低耦合 

#  线程池的实现原理

##  池化技术

一说到线程池自然就会想到 **池化技术** 。

其实所谓池化技术，就是把一些能够复用的东西放到池中，避免重复创建、销毁的开销，从而极大提高性能。

常见池化技术的例如：

  * 线程池 
  * 内存池 
  * 连接池 

##  Java中的实现

###  官方接口

JDK 1.5 推出了三大API用来创建线程：

  * ` Executors.newCachedThreadPool() ` ：无限线程池（最大21亿） 
  * ` Executors.newFixedThreadPool(nThreads) ` ：固定大小的线程池 
  * ` Executors.newSingleThreadExecutor() ` ：单个线程的线程池 

这三个API的底层其实都是由同一个类实现的： ` ThreadPoolExecutor ` 类

    
    
    public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
    }
    
    
    
    public static ExecutorService newFixedThreadPool(int nThreads) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>());
    }
    
    
    
    public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
    }
    

###  ` ThreadPoolExecutor ` 类

####  七大参数

` ThreadPoolExecutor ` 类主要有以下七个参数：

  * ` int corePoolSize ` : 核心线程池大小 
  * ` int maximumPoolSize ` : 最大核心线程池大小 
  * ` long keepAliveTime ` : 线程空闲后的存活时间 
  * ` TimeUnit unit ` : 超时单位 
  * ` BlockingQueue<Runnable> workQueue ` : 阻塞队列 
  * ` ThreadFactory threadFactory ` : 线程工厂：创建线程的，一般默认 
  * ` RejectedExecutionHandler handle ` : 拒绝策略 

####  四种拒绝策略

拒绝策略就是当队列满时，线程如何去处理新来的任务。

Java内置了四种拒绝策略：

  * ` ThreadPoolExecutor.CallerRunsPolicy() `

  * ` ThreadPoolExecutor.AbortPolicy() `

  * ` ThreadPoolExecutor.DiscardPolicy() `

  * ` ThreadPoolExecutor.DiscardOldestPolicy() `

#####  CallerRunsPolicy（调用者运行策略）

**功能** ：只要线程池没有关闭，就由提交任务的当前线程处理。

**使用场景** ：一般在不允许失败、对性能要求不高、并发量较小的场景下使用。

#####  AbortPolicy（中止策略）

**功能：** 当触发拒绝策略时，直接抛出拒绝执行的异常

**使用场景：** ` ThreadPoolExecutor ` 中默认的策略就是 ` AbortPolicy ` ，由于 `
ExecutorService ` 接口的系列 ` ThreadPoolExecutor ` 都没有显示的设置拒绝策略，所以默认的都是这个。

#####  DiscardPolicy（丢弃策略）

**功能：** 直接丢弃这个任务，不触发任何动作

**使用场景：** 提交的任务无关紧要，一般用的少。

#####  DiscardOldestPolicy（弃老策略）

**功能：** 弹出队列头部的元素，然后尝试执行，相当于排队的时候把第一个人打死，然后自己代替

**使用场景：** 发布消息、修改消息类似场景。当老消息还未执行，此时新的消息又来了，这时未执行的消息的版本比现在提交的消息版本要低就可以被丢弃了。

#  线程池中的状态

![img](https://user-gold-
cdn.xitu.io/2020/7/8/1732e1655d5f300a?imageView2/0/w/1280/h/960/format/webp/ignore-
error/1)

  * ` RUNNING ` 自然是运行状态，指可以接受任务执行队列里的任务 
  * ` SHUTDOWN ` 指调用了 ` shutdown() ` 方法，不再接受新任务了，但是队列里的任务得执行完毕。 
  * ` STOP ` 指调用了 ` shutdownNow() ` 方法，不再接受新任务，同时抛弃阻塞队列里的所有任务并中断所有正在执行任务。 
  * ` TIDYING ` 所有任务都执行完毕，在调用 ` shutdown()/shutdownNow() ` 中都会尝试更新为这个状态。 
  * ` TERMINATED ` 终止状态，当执行 ` terminated() ` 后会更新为这个状态。 

#  处理流程

提交一个任务到线程池中，线程池的处理流程如下：

  * 判断线程池里的 **核心线程** 是否都在执行任务 

    * 是：进入下个流程。 
    * 否：调用/创建一个 **新的核心线程** 来执行任务。 
  * 线程池判断工作队列是否已满 

    * 是：进入下个流程。 
    * 否：将新提交的任务存储在这个工作队列里。 
  * 判断 **线程池里的所有线程** 是否都处于工作状态 

    * 是：交给拒绝策略来处理这个任务。 
    * 否：调用/创建一个新的工作线程来执行任务。 

#  具体使用

##  创建

不过最好 **不要使用** ` Executors ` 来创建线程，原因如下（参考自——阿里巴巴Java开发手册）：

  * ` FixedThreadPool ` 和 ` SingleThreadPool ` ： 允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM 
  * ` CachedThreadPool ` ： 允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM 

推荐使用 ` ThreadPoolExecutor ` 类自行创建

    
    
    // 自定义线程池
    ExecutorService threadPool = new ThreadPoolExecutor(
      2,
      Runtime.getRuntime().availableProcessors(),//CPU的核心数，适合CPU密集型任务
      3,
      TimeUnit.SECONDS,
      new LinkedBlockingDeque<>(3),
      Executors.defaultThreadFactory(),
      new ThreadPoolExecutor.DiscardOldestPolicy());
    

###  合理配置线程

线程池不是越大越好，要根据任务类型合理进行配置

  * IO 密集型任务：尽可能的多配置线程 
  * CPU 密集型任务：（大量复杂的运算）应当分配较少的线程 

##  执行

有两个方法可以执行任务 ` execute ` 和 ` submit `

  * ` execute ` 提交没有返回值，不能判断是否执行成功。 
  * ` submit ` 会返回一个 ` Future ` 对象，通过 ` Future ` 的 ` get() ` 方法来获取返回值。 

区别：

  * 接收的参数不一样 
    * ` execute ` 提交的方式只能提交一个Runnable的对象 
    * ` submit ` 有三种 
  * ` submit ` 有返回值，而 ` execute ` 没有 
  * ` submit ` 方便 ` Exception ` 处理 
  * ` execute ` 是 ` Executor ` 接口中唯一定义的方法 
  * ` submit ` 是 ` ExecutorService ` （该接口继承 ` Executor ` ）中定义的方法 

##  关闭

线程池使用完毕，需要对其进行关闭，有两种方法

  * ` shutdown() ` ：不再继续接收新的任务，执行完成已有任务后关闭 

  * ` shutdownNow() ` ：直接关闭，若果有任务尝试停止 

