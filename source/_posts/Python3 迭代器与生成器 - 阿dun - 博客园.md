---
title: Python3 迭代器与生成器
date: 2020-01-15 17:08
categories: ['Python']
tags: ['迭代器', '生成器', 'python', '协程']
---
#  可迭代对象（Iterable）

  * **包含**

    * 迭代器 
      * 生成器 
    * 序列 
      * list 
      * str 
      * tuple 
    * 字典 
  * **迭代器协议**

对象需要提供next()方法，它要么返回迭代中的下一项，要么就引起一个StopIteration异常，以终止迭代。

#  迭代器（Iterator）

##  定义

  * 有两个基本的方法：iter() 和 next()。 
  * 访问集合元素的一种方式。 
  * 可以记住遍历的位置的对象。 
  * 迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不能后退。 

##  迭代器和可迭代对象的区别

  * 可迭代对象 **包含** 迭代器。 
  * 如果一个对象拥有 ` __iter__() ` 方法，就是可迭代对象。 
  * 定义迭代器，必须在实现 ` __iter__() ` 的基础上，再实现 ` next() ` 方法。 
  * 可迭代对象可被 ` for ` 遍历，迭代器在此基础上可以调用 ` next() ` 方法 

##  创建一个迭代器

###  创建一个迭代器类

把一个类作为一个迭代器使用需要在类中实现两个方法 ` __iter__() ` 与 ` __next__() ` 。

  * **iter** () 方法返回一个特殊的迭代器对象， 这个迭代器对象实现了 **next** () 方法并通过 StopIteration 异常标识迭代的完成。 
  * **next** () 方法（Python 2 里是 next()）会返回下一个迭代器对象。 

_例：_

    
    
    from collections import Iterable,Iterator
    
    #创建一个返回数字的迭代器, 初始值为 1, 逐步递增1 
    class MyNumbers:
        def __init__(self):
            self.a = 1  
    
        def __iter__(self):
            return self
    
        def __next__(self):
            x = self.a
            self.a += 1
            return x
    
    a = MyNumbers()
    print(isinstance(a, Iterator))#判断是否是迭代器
    print(next(a))
    print(next(a))
    print(next(a))
    print(next(a))
    print(next(a))
    
    

**运行结果如下：**

    
    
    True
    1
    2
    3
    4
    5
    

###  使用内置 ` iter() ` 函数

使用 ` iter() ` 函数将可迭代对象转换为迭代器  
**例：**

    
    
    from collections import Iterable,Iterator
    a = [1,2,3,4]
    b = iter(a)
    print(type(a))#查看类型
    print(type(b))
    print(isinstance(a, Iterable))#判断是否为可迭代对象
    print(isinstance(b, Iterable))
    print(isinstance(a, Iterator))#判断是否是迭代器
    print(isinstance(b, Iterator))
    
    print(next(b))
    print(next(b))
    print(next(b))
    print(next(b))
    

**运行结果如下:**

    
    
    <class 'list'>
    <class 'list_iterator'>
    True
    True
    False
    True
    1
    2
    3
    4
    

##  StopIteration异常

StopIteration 异常用于标识迭代的完成，防止出现无限循环的情况，在 ` __next__() ` 方法中我们可以设置在完成指定循环次数后触发
StopIteration 异常来结束迭代。

**例：在 10 次迭代后停止执行**

    
    
    class MyNumbers:
        def __init__(self):
            self.a = 1
    
        def __iter__(self):
            return self
    
        def __next__(self):
            if self.a <= 10:
                x = self.a
                self.a += 1
                return x
            else:
                raise StopIteration
     
    a = MyNumbers()
    for i in a:
        print(i)
    

**执行结果如下**

    
    
    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    

#  生成器（generator）

##  定义

生成器是一种特殊的迭代器，生成器自动实现了“迭代器协议”（即__iter__和next方法），不需要再手动实现两方法。

生成器在迭代的过程中可以改变当前迭代值，而修改普通迭代器的当前迭代值往往会发生异常，影响程序的执行。

##  Python有两种不同的方式提供生成器

###  生成器函数：

常规函数定义，但是，使用yield语句而不是return语句返回结果。yield语句一次返回一个结果，在每个结果中间，挂起函数的状态，以便下次重它离开的地方继续执行

**例：**

    
    
    def abc():
        yield "a"
        yield "b"
        yield "c"
    
    
    x = abc()
    print(next(x))
    print(next(x))
    print(next(x))
    

**执行结果如下：**

    
    
    a
    b
    c
    

###  生成器表达式：

类似于列表推导，但是，生成器返回按需产生结果的一个对象，而不是一次构建一个结果列表

    
    
    a = [i for i in range(5)]#列表推导
    b = (i for i in range(5))#生成器表达式
    print(type(a))
    print(type(b))
    
    

**执行结果如下：**

    
    
    <class 'list'>
    <class 'generator'>
    

##  特点

###  语法上和函数类似：

生成器函数和常规函数几乎是一样的。它们都是使用def语句进行定义。差别在于，生成器使用yield语句返回一个值，而常规函数使用 ` return `
语句返回一个值

###  自动实现迭代器协议：

对于生成器，Python会自动实现迭代器协议，以便应用到迭代背景中（如for循环，sum函数）。由于生成器自动实现了迭代器协议，所以，我们可以调用它的 `
next() ` 方法，并且，在没有值可以返回的时候，生成器自动产生StopIteration异常

###  状态挂起：

生成器使用yield语句返回一个值。yield语句挂起该生成器函数的状态，保留足够的信息，以便之后从它离开的地方继续执行。

###  延迟计算:

一次返回一个结果。不会一次生成所有的结果，在对于大数据量处理时，不一会因为一次性生成大量结果导致机器直接卡死。

###  生成器只能遍历一次:

**例：**

    
    
    a = (x ** 2 for x in range(4)) 
    for i in a:
        print(i)
    for i in a:
        print(i)
    else:
        print("再次遍历什么都没有输出")
    

**执行结果如下：**

    
    
    0
    1
    4
    9
    再次遍历什么都没有输出
    

* * *

