---
title: Python3 多进程编程(Multiprocess programming)
date: 2019-10-16 22:26
categories: ['Python']
tags: ['多线程', 'python', '学习笔记', '多进程']
---
#  Python3 多进程编程(Multiprocess programming)

##  为什么使用多进程

python中的多线程其实并不是真正的多线程，不能充分地使用多核CPU的资源，此时需要使用需要使用多进程解决问题。

##  具体用法

Python中的多进程是通过 ` multiprocessing ` 模块来实现的，和多线程的 ` threading.Thread ` 类似，利用 `
multiprocessing.Process ` 来创建一个进程对象。进程对象的方法和线程对象的方法类似，也有start(), join()等。

  * **直接启用**

代码实例

    
        import multiprocessing
    from time import sleep
    
    def clock(interval):
        i = 0
        while i<5:
            i += 1
            print(f"Run >>> {i}")
            sleep(interval)
        print("Ending!")
    
    
    if __name__ == '__main__':
        p = multiprocessing.Process(target = clock, args = (1,))
        p.start()
        p.join()
    
    

运行结果

    
        Run >>> 1
    Run >>> 2
    Run >>> 3
    Run >>> 4
    Run >>> 5
    Ending!
    

  * **继承自` multiprocessing.Process ` 调用 **

与多线程使用方法类似

    * 直接继承Process 
    * 重写run函数 
    * 类实例可以直接运行 
代码实例

    
        import multiprocessing
    from time import sleep
    
    class ClockProcess(multiprocessing.Process): 
        def __init__(self, interval):
            super().__init__()
            self.interval = interval
    
        def run(self):
            i = 0
            while i<5:
                i += 1
                print(f"Run >>> {i}")
                sleep(self.interval)
            print("Ending!")
    
    
    if __name__ == '__main__':
        p = ClockProcess(1)
        p.start()
        p.join()
    
    

运行结果

    
            Run >>> 1
        Run >>> 2
        Run >>> 3
        Run >>> 4
        Run >>> 5
        Ending!
    

  * **守护进程**

    * 设置该进程为守护进程，即认为此进程不重要，主进程结束后，该进程随即结束。 
    * 用法 ` Process.daemon = Ture `
未使用守护进程

    
        import multiprocessing
    from time import sleep
    
    def clock(interval):
        i = 0
        while i<5:
            i += 1
            print(f"Run >>> {i}")
            sleep(interval)
        print("Ending!")
    
    def run():
        p = multiprocessing.Process(target = clock, args = (1,))
        p.start()
    
    if __name__ == '__main__':
        run()
        sleep(2)
        print("ENDING!")
    

运行结果

    
        Run >>> 1
    Run >>> 2
    ENDING!
    Run >>> 3
    Run >>> 4
    Run >>> 5
    Ending!
    

使用守护进程

    
        import multiprocessing
    from time import sleep
    
    def clock(interval):
        i = 0
        while i<5:
            i += 1
            print(f"Run >>> {i}")
            sleep(interval)
        print("Ending!")
    
    def run():
        p = multiprocessing.Process(target = clock, args = (1,))
        p.daemon = True
        p.start()
    
    if __name__ == '__main__':
        run()
        sleep(2)
        print("ENDING!")
    

运行结果

    
        Run >>> 1
    Run >>> 2
    ENDING!
    

#  Python多线程的通信

进程是系统独立调度核分配系统资源的基本单位，进程之间是相互独立的，进程之间的数据也不能共享，这是多进程在使用中与多线程最明显的区别。  
  
所以要使用多进程来弥补Python中多线程的不足，解决多进程之间的通信时关键。

##  进程对列Queue

在python多进程中，Queue其实就是进程之间的数据管道，实现进程通信。

###  生产者消费者问题

  * 仓库(固定大小的中间缓冲区) 

  * 生产者 

持续生产数据传入仓库

  * 消费者 

持续从仓库总提取数据

在实际运行时会产设的问题。生产者的主要作用是生成一定量的数据放到仓库中，然后重复此过程。  
  
与此同时，消费者也在仓库消耗这些数据。该问题的关键就是要保证生产者不会在仓库满时加入数据，消费者也不会在仓库空时消耗数据。

###  JoinableQueue

  * JoinableQueue同样通过multiprocessing使用。 

  * JoinableQueue([maxsize])：就是一个Queue对象，但允许项目的使用者通知生成者项目已经被成功处理。 

  * maxsize是队列中允许最大项数，省略则无大小限制。 

  * 方法介绍： 

JoinableQueue与Queue对象的方法一致，且之外还具有：

    * ` JoinableQueue.task_done() ` ：使用者使用此方法发出信号，表示q.get()的返回项目已经被处理。如果调用此方法的次数大于从队列中删除项目的数量，将引发ValueError异常。 
    * ` JoinableQueue.join() ` :生产者调用此方法进行阻塞，直到队列中所有的项目均被处理。阻塞将持续到队列中的每个项目均调用 ` JoinableQueue.task_done() ` 方法为止。 

###  Queue实例

  * 使用JoinableQueue实现生产者消费者模型 

代码

    
        import multiprocessing
    from time import ctime,sleep
    
    def consumer(input_q):
        print("消费开始:", ctime())
        while True:
            # 处理项
            item = input_q.get()
            print ("消费 >>>>>>>>>", item) # 此处替换为有用的工作
            input_q.task_done() # 发出信号通知任务完成
            sleep(1)
        print ("消费结束:", ctime()) ##此句未执行，因为q.join()收集到四个task_done()信号后，主进程启动，未等到print此句完成，程序就结束了
    
    
    def producer(sequence, output_q):
        print ("生产开始:", ctime())
        for item in sequence:
            output_q.put(item)
            print ("生产 >>>>>>>>>", item)
            sleep(1)
        print ("生产结束:", ctime())
    
    if __name__ == '__main__':
        q = multiprocessing.JoinableQueue()
        # 运行消费者进程
        cons_p = multiprocessing.Process (target = consumer, args = (q,))
        cons_p.daemon = True
        cons_p.start()
    
        # 生产多个项，sequence代表要发送给消费者的项序列
        # 在实践中，这可能是生成器的输出或通过一些其他方式生产出来
        sequence = [1,2,3,4]
        producer(sequence, q)
        # 等待所有项被处理
        q.join()
    

运行结果

    
        生产开始: Wed Oct 16 22:06:11 2019
    生产 >>>>>>>>> 1
    消费开始: Wed Oct 16 22:06:11 2019
    消费 >>>>>>>>> 1
    生产 >>>>>>>>> 2
    消费 >>>>>>>>> 2
    生产 >>>>>>>>> 3
    消费 >>>>>>>>> 3
    生产 >>>>>>>>> 4
    消费 >>>>>>>>> 4
    生产结束: Wed Oct 16 22:06:15 2019
    

##  管道Pipe

待续

