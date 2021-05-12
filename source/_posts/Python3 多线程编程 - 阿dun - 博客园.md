---
title: Python3 多线程编程
date: 2019-10-11 22:00
categories: ['Python']
tags: ['python', '多线程', '学习笔记']
---
#  线程

##  什么是线程

  * **官方定义：**

线程（thread）是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。

  * **说人话：**

假如 **进程** 是保洁公司， **线程**
就是公司的员工。当公司接到任务时，干活的是员工，而进程负责分配员工。可以好几个员工擦一块玻璃，也可以一个员工收拾一个屋子。

##  特点

  * **独立调度和分派的基本单位**

一个公司里至少得有一个员工，才能干活。公司分配各项工作给团队，最后团队还是会分配给个人。

  * **轻型实体**

每个线程占用的系统资源非常少。

  * **可并发执行**

多个线程可以同时工作，就像公司里的员工可以一切干活。

  * **共享进程资源**

所有的线程共享该进程所拥有的资源。

**例如：** 所有线程地址空间都相同(进程的地址空间)，就好比一家公司的所有员工的工作地址都为公司的所在地。

##  线程与进程的关系

继续用保洁公司举例子

  * 公司本身是进程，公司里又很多员工，每个员工都是一个线程，同时公司里还有很多共享的资源，比如：扫把、墩布、毛巾、饮水机等。 
  * 但是与现实模型不同的是，这些人是由多个 cpu 控制的，例如 4 个 cpu 对应 40 个人，cpu 需要切换控制。 
  * 真正执行工作的是公司员工,也就是进程的任务靠线程执行 
  * 这些人可以并行工作，处理事情。也就是多线程可以同时运行。 
  * 当大家访问公共资源的时候会有冲突，例如：都要出去擦玻璃，就剩一条毛巾了，这时就需要排队。在进程中就叫做上锁。 

* * *

#  Python3中的多线程

##  全局解释器锁（GIL）

###  GIL是啥？

  * GIL并不是Python语言的特性，它是在现实Python解释器时引用的一个概念。 

  * GIL只在CPython解释器上存在。作用是保证同一时间内只有一个线程在执行。 

  * 解决解释器中多个线程的竞争资源问题 

###  GIL对Python程序有啥影响？

  * Python中同一时刻有且只有一个线程会执行。 

因此,Python中的多线程并不算是真正意义上的多线程。

  * Python中的多个线程由于GIL锁的存在无法利用多核CPU。 

因此,Python中的多线程不适合计算机密集型的程序。

    * 计算密集型程序 

计算密集型又叫CUP密集型，这类程序绝大部分运行时间都消耗在CPU计算上，这个时候，不论你开多少线程，他用的时间都是这么多，甚至比原来时间还长，因为GIL一个时刻只让你执行一个线程，大部分计算密集型任务你分了很多线程但是依然会按照代码顺序线性执行。分很多线程没有什么改善，反而因为代码的冗杂可能更加慢。

    * IO密集型程序 

IO密集型顾名思义就是主要进行I/O操作，90%以上的时间都花费在网络、硬盘、输入输出上了，CPU执行完命令之后其实就没事干了，就可以释放内存来执行下一条命令了，不用让CPU在那干等着，这样就能大大提高程序的运行效率。

###  改善GIL产生的问题

  * 使用更高版本的解释器，优化对Python的解释 
  * 变更解释器，如(JPython)，但可能因为相对小众，支持的模块相对较少，开发效率变低 
  * 用多进程方案替代多线程 

##  Python3关于多线程的模块

Python的标准库提供了两个模块：_thread和threading

  * _thread 

在Python3之前为thread，有于存在缺陷，不建议使用，在Python3中封装成_thread

  * threading 

thread的继承者，绝大多数情况下使用threading就够了。

##  多线程使用

  * **直接调用**

把一个函数传入创建Thread实例，然后调用start()开始执行

    
        import threading
    import time
    def loop():
        #threading.current_thread().name获取当前线程的名字
        print(f"线程（ {threading.current_thread().name} ）正在执行……")
        num = 0
        while num < 5:
            num += 1
            print(f"线程（ {threading.current_thread().name} ）>>> {num}")
            time.sleep(1)
        print(f"线程（ {threading.current_thread().name}） 执行结束")
    
    if __name__ == '__main__':
        print(f"线程（ {threading.current_thread().name} ）正在执行……")
        # 创建线程实体，target导入的函数，name线程的名字，初始为Tread1，之后类推2，3，4
        # 如果导入函数有参数，使用 args=() 添加函数参数
        t = threading.Thread(target=loop,name='LoopThread')
        # 启动线程
        t.start()
        # 等待多线程结束
        t.join()
        print(f"线程（ {threading.current_thread().name} ）执行结束")
    
    

执行结果如下：

    
        线程（ MainThread ）正在执行……
    线程（ LoopThread ）正在执行……
    线程（ LoopThread ）>>> 1
    线程（ LoopThread ）>>> 2
    线程（ LoopThread ）>>> 3
    线程（ LoopThread ）>>> 4
    线程（ LoopThread ）>>> 5
    线程（ LoopThread） 执行结束
    线程（ MainThread ）执行结束
    

  * **继承自threading.Thread调用**

    * 直接继承Thread 
    * 重写run函数，run函数代表的是真正执行的功能 
    * 类实例可以直接运行 
    
        import threading
    import time
    # 1. 类需要继承自threading.Thread
    class MyThread(threading.Thread):
        def __init__(self, arg):
            super(MyThread, self).__init__()
            self.arg = arg
    
        # 2 必须重写run函数，run函数代表的是真正执行的功能
        def  run(self):
            time.sleep(2)
            print(f"Run  >>>  {self.arg}")
    
    for i in range(1,4):
        t = MyThread(i)
        t.start()
        t.join()
    
    print("End")
    
    

执行结果如下：

    
        Run  >>>  1
    Run  >>>  2
    Run  >>>  3
    End
    

  * **守护线程**

    * 不设置守护线程 
        
                import time
        import threading
        
        def fun():
            print("启动Fun")
            time.sleep(2)
            print("结束Fun")
        
        print("启动Main")
        t = threading.Thread(target=fun)
        t.start()
        time.sleep(1)
        print("结束Main")
        

运行结果

        
                启动Main
        启动Fun
        结束Main
        结束Fun
        

    * 设置守护线程 
        
                import time
        import threading
        
        def fun():
            print("启动Fun")
            time.sleep(2)
            print("结束Fun")
        
        print("启动Main")
        t = threading.Thread(target=fun)
        t.setDaemon(True)
        t.start()
        time.sleep(1)
        print("结束Main")
        

运行结果

        
                启动Main
        启动Fun
        结束Main
        

如果你设置一个线程为守护线程，，就表示你认为此线程不重要，在进程退出的时候，不用等待这个线程即可退出。

` thread.setDaemon(True|False) ` 表示此线程是否为守护线程。但此语句必须加在 ` thread.start() ` 之前

  * 常用函数 

    * ` threading.enumerate() ` : 

返回一个包含正在运行的线程的list

    * ` threading.activeCount() ` : 

返回正在运行的线程数量，效果跟 len(threading.enumerate)相同

    * ` thr.setName() ` : 给线程设置名字 

` thr.getName() ` : 得到线程的名字

      * thr表示线程实例，如 ` t1.getName() `

      * ` thr.getName() ` 获取指定线程的名字， ` threading.current_thread().name ` 获取当前线程的名字 

##  共享变量

多线程同时访问同一变量时，会产生共享变量的问题，造成变量冲突产生问题。

  * 实例 

执行多线程，对同一变量进行加减操作，代码如下。

    
        import threading
    
    sum = 0
    loopSum = 1000000
    
    def myAdd():
        global  sum, loopSum
        for i in range(1, loopSum):
            sum += 1
    
    def myMinu():
        global  sum, loopSum
        for i in range(1, loopSum):
            sum -= 1
    
    if __name__ == '__main__':
        print(f"Starting ....{sum}")
    
        t1 = threading.Thread(target=myAdd, args=())
        t2 = threading.Thread(target=myMinu, args=())
    
        t1.start()
        t2.start()
    
        t1.join()
        t2.join()
    
        print(f"Done .... {sum}")
      
    

两次分别如下执行结果如下：

    
        Starting ....0
    Done .... -278763
    
    
        Starting ....0
    Done .... 662850
    

可以发现，运算结果与我们传统认知不同。原因是Python在内部实现运算时是一个复杂的过程，同时对 ` sum `
进行修改时产生的相互干扰，故出现未知错误，导致结果出错。

  * 解决方法：上锁 

**锁(Lock)** ：是一个标志，表示一个线程在占用一些共享资源.

    * 那些资源需要上锁？ 

需要被共享使用的，可能产生使用冲突的

    * 注意： 

避免产生死锁，导致程序陷入死循环

    * 线程安全问题： 

      * 如果一个资源，他对于多线程来讲，不用加锁也不会引起任何问题，则称为线程安全。 
      * 线程不安全变量类型： list, set, dict 
      * 线程安全变量类型： queue 

**使用案例，代码如下：**

    
        import threading
    
    sum = 0
    loopSum = 1000000
    lock = threading.Lock()
    def myAdd():
        global  sum, loopSum
        for i in range(1, loopSum):
            # 上锁，申请锁
            lock.acquire()
            sum += 1
            # 释放锁
            lock.release()
    
    def myMinu():
        global  sum, loopSum
    
        for i in range(1, loopSum):
            lock.acquire()
            sum -= 1
            lock.release()
    
    if __name__ == '__main__':
        print(f"Starting ....{sum}")
    
        t1 = threading.Thread(target=myAdd, args=())
        t2 = threading.Thread(target=myMinu, args=())
    
        t1.start()
        t2.start()
    
        t1.join()
        t2.join()
    
        print(f"Done .... {sum}")
    

执行结果：

    
        Starting ....0
    Done .... 0
    

