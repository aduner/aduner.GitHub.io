---
title: Python3 异步神器-协程(Coroutine)
date: 2020-01-16 22:39
categories: ['Python']
tags: ['协程', 'python']
---
#  什么是协程

协程(Coroutine)，又称微线程，纤程。通常我们认为线程是轻量级的进程，因此我们也把协程理解为轻量级的线程即微线程。

协程的作用是在执行函数A时可以随时中断去执行函数B，然后中断函数B继续执行函数A（可以自由切换）。这里的中断，不是函数的调用，而是有点类似CPU的中断。这一整个过程看似像多线程，然而协程只有一个线程执行。

#  协程的优势

  * 执行效率极高，因为是子程序（函数）切换不是线程切换，由程序自身控制，没有切换线程的开销。所以与多线程相比，线程的数量越多，协程的性能优势越明显。 
  * 不需要 **锁** 机制，因为只有一个线程，也不存在同时写变量冲突，在控制共享资源时也不需要加锁，只需要判断状态，因此执行效率高的多。 
  * 协程可以处理IO密集型程序的效率问题，但不适合处理CPU密集型程序，如要充分发挥CPU利用率应结合多进程+协程。 

#  Python3中的协程

##  生成器 yield/send

###  yield + send（利用生成器实现协程）

**例:**

    
    
    def simple_coroutine():
        print('-> start')
        x = yield
        print(x)
        print('-> end')
    
    #主线程
    sc = simple_coroutine()
    # 可以使用sc.send(None)，效果一样
    next(sc) #预激
    sc.send('go')
    

**执行结果如下：**

    
    
    -> start
    go
    -> end
    
    抛出StopIteration
    

  * ` simple_coroutine() ` 是一个生成器，由 ` next(sc) ` 预激，启动协程，执行到第一个yield中断 
  * ` send() ` 方法给yield传入参数，继续执行 

###  协程的四个状态

  * GEN_CREATED：等待开始执行 
  * GEN_RUNNING：解释器正在执行 
  * GEN_SUSPENED：在yield表达式处暂停 
  * GEN_CLOSED：执行结束 

###  协程终止

  * 协程中未处理的异常会向上冒泡，传给 next 函数或 send 方法的调用方（即触发协程的对象） 
  * 终止协程的一种方式：发送某个哨符值，让协程退出。内置的 None 和Ellipsis 等常量经常用作哨符值 

##  @asyncio.coroutine和yield from

###  asyncio.coroutione

` asyncio ` 是Python3.4版本引入的标准库，直接内置了对异步IO的支持。 ` asyncio ` 的异步操作，需要在 `
coroutine ` 中通过 ` yield from ` 完成。  
**例：**

    
    
    import asyncio
    @asyncio.coroutine
    def test(i):
        print('test_1', i)
        r = yield from asyncio.sleep(1)
        print('test_2', i)
    
    loop = asyncio.get_event_loop()
    tasks = [test(i) for i in range(1,3)]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()
    

  * ` @asyncio.coroutine ` 把一个 ` generator ` 标记为 ` coroutine ` 类型，然后就把这个 ` coroutine ` 扔到 ` EventLoop ` 中执行。 ` test() ` 会首先打印出test_1 
  * 然后 ` yield from ` 语法可以让我们方便地调用另一个 ` generator ` 。由于 ` asyncio.sleep() ` 也是一个 ` coroutine ` ，所以线程不会等待 ` asyncio.sleep() ` ，而是直接中断并执行下一个消息循环。 
  * 当 ` asyncio.sleep() ` 返回时，线程就可以从 ` yield from ` 拿到返回值（此处是None） 
  * 然后接着执行下一行语句。把 ` asyncio.sleep(1) ` 看成是一个耗时1秒的IO操作，在此期间主线程并未等待，而是去执行 ` EventLoop ` 中其他可以执行的 ` coroutine ` 了，因此可以实现并发执行 

**结果如下：**

    
    
    test_1 1
    test_1 2
    test_2 1
    test_2 2
    

###  yield from

当 ` yield from ` 后面加上一个生成器后，就实现了生成的嵌套。  
实现生成器的嵌套，并不是一定必须要使用yield from，而是使用yield
from可以让我们避免让我们自己处理各种料想不到的异常，而让我们专注于业务代码的实现。  
如果自己用yield去实现，那只会加大代码的编写难度，降低开发效率，降低代码的可读性。

  * **调用方** ：调用委派生成器的客户端（调用方）代码 
  * **委托生成器** ：包含yield from表达式的生成器函数 
  * **子生成器** ：yield from后面加的生成器函数 

    
    
    # 子生成器
    def average_gen():
        total = 0
        count = 0
        average = 0
        while True:
            new_num = yield average
            count += 1
            total += new_num
            average = total/count
    
    # 委托生成器
    def proxy_gen():
        while True:
            yield from average_gen()
    
    # 调用方
    def main():
        calc_average = proxy_gen()
        next(calc_average)# 预激下生成器
        print(calc_average.send(10))  
        print(calc_average.send(20))  
        print(calc_average.send(30))  
    
    main()
    
    

结果如下：

    
    
    10.0
    15.0
    20.0
    

**委托生成器的作用：**  
在调用方与子生成器之间建立一个双向通道。

**双向通道** 就是调用方可以通过 ` send() ` 直接发送消息给子生成器，而子生成器yield的值，也是直接返回给调用方。

###  为什么要用yield from

yield
from帮我们做了很多的异常处理，而这些如果我们要自己去实现的话，一个是编写代码难度增加，写出来的代码可读性很差，很可能有遗漏，只要哪个异常没考虑到，都有可能导致程序崩溃。

##  async/await关键字

为了简化并更好地标识异步IO，从Python 3.5开始引入了新的语法async和await，可以让coroutine的代码更简洁易读。

可将上面的案例代码改造如下：

    
    
    import asyncio
    # @asyncio.coroutine
    async def test(i):
        print('test_1', i)
        # r = yield from asyncio.sleep(1)
        await asyncio.sleep(1)
        print('test_2', i)
    
    loop = asyncio.get_event_loop()
    tasks = [test(i) for i in range(1,3)]
    loop.run_until_complete(asyncio.wait(tasks))
    loop.close()
    

执行结果

    
    
    test_1 1
    test_1 2
    test_2 1
    test_2 2
    

  * ` async ` 替换 ` @asyncio.coroutine ` ，加在def之前用于修饰 
  * ` await ` 替换 ` yield from `

* * *

