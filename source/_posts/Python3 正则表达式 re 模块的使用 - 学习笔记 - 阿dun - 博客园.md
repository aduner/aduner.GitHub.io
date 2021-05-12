---
title: Python3 正则表达式 re 模块的使用 - 学习笔记
date: 2020-01-31 20:04
categories: ['Python']
tags: ['正则表达式', 'python']
---
#  re 模块的引入

>   * Python 自1.5版本起增加了 ` re ` 模块，它提供 Perl 风格的正则表达式模式。
>
>   * ` re ` 模块使 Python 语言拥有全部的正则表达式功能。
>
>

#  re 模块的使用

> **参数含义**
>
>   * pattern: 字符串形式的正则表达式
>   * string: 要匹配的字符串
>   * flags: 可选，表示匹配模式
>   * pos:可选，字符串中开始搜索的位置索引
>   * endpos:可选，endpos 限定了字符串搜索的结束
>   * 不填pos endpos默认扫描全部
>

##  re.compile()

> compile(pattern, flags=0)

  * 将正则表达式的样式编译为一个 正则表达式对象 （正则对象） 
  * 可以使用正则对象调用 ` match() ` 等函数 

    
    
    >>> test = '1 one 2 two 3 three'
    >>> a=re.compile(r'\d+')
    >>> b=a.match(test)
    >>> print(f"输出：{b[0]}")
    
    
    输出：1
    

##  re.match()与re.search()

###  re.match

> re.match(pattern, string, flags=0)
>
> Pattern.match(string, pos, endpos)

  * 如果 string 的 **开始位置** 能够找到这个正则样式的任意个匹配，就返回一个相应的 匹配对象。如果不匹配，就返回 None 
  * 可以使用 ` group(num) ` 或 ` groups() ` 匹配对象函数来获取匹配表达式 
    * ` group(num=0) ` 表示匹配的整个表达式的字符串 
    * ` group() ` 可以一次输入多个 **组号** ，在这种情况下它将返回一个包含那些组所对应值的元组。 
    * ` groups() ` 返回一个包含所有小组字符串的元组，从 1 到 所含的小组号。 

    
    
    >>> test = '1 one 2 two 3 three'
    >>> a=re.compile(r'(\d+) (\w+)')
    >>> b=a.match(test)
    >>> print(f"输出：{b.group()}")
    >>> print(f"输出：{b.group(2)}")
    >>> print(f"输出：{b.group(1,2)}")
    >>> print(f"输出：{b.groups()}")
    
    
    输出：1 one
    输出：one
    输出：('1', 'one')
    输出：('1', 'one')
    

  * ` Match.start([group]) ` 和 ` Match.end([group]) `
    * 返回 ` group ` 匹配到的字串的开始和结束标号。 
    * 如果 ` group ` 存在，但未产生匹配，就返回 ` -1 ` 。 
  * ` Match.span([group]) `
    * 对于一个匹配 m ，返回一个二元组 ` (m.start(group), m.end(group)) `
    * 注意如果 ` group ` 没有在这个匹配中，就返回 ` (-1, -1) `

###  re.search()

> re.search(pattern, string, flags=0)
>
> Pattern.search(string, pos, endpos)

  * **扫描整个** string 寻找第一个匹配的位置， 并返回一个相应的 匹配对象。如果没有匹配，就返回 None 
  * 其他与 ` match() ` 一致 

    
    
    >>> test = 'one 2 two 3 three'
    >>> a = re.compile(r'(\d+) (\w+)')
    >>> b = a.search(test)
    >>> c = a.match(test)
    >>> print(c)
    >>> print(f"输出：{b.group()}")
    >>> print(f"输出：{b.group(2)}")
    >>> print(f"输出：{b.group(1,2)}")
    >>> print(f"输出：{b.groups()}")
    
    
    输出：None
    输出：2 two
    输出：two
    输出：('2', 'two')
    输出：('2', 'two')
    

###  区别

  * ` match() ` 只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回 None 
  * 而 ` search() ` 匹配整个字符串，直到找到一个匹配 

##  re.findall()与re.finditer()

###  re.findall()

> re.findall(pattern, string, flags=0)
>
> Pattern.findall(string, pos, endpos)

  * 对 string 返回一个不重复的 pattern 的匹配列表， string 从左到右进行扫描，匹配按找到的顺序返回 
  * 如果样式里存在一到多个组，就返回一个组合列表；就是一个元组的列表（如果样式里有超过一个组合的话） 

    
    
    >>> test = 'one 2 two 3 three'
    >>> a = re.compile(r'(\d+) (\w+)')
    >>> b = a.search(test)
    >>> b=a.findall(test)
    >>> print(f"输出：{b}")
    
    输出：[('2', 'two'), ('3', 'three')]
    

###  re.finditer()

> re.finditer(pattern, string, flags=0)
>
> Pattern.finditer(string, pos, endpos)

  * pattern 在 string 里所有的非重复匹配，返回为一个 **迭代器** iterator 保存了 匹配对象 

    
    
    >>> test = 'one 2 two 3 three'
    >>> a = re.compile(r'(\d+) (\w+)')
    >>> b = a.finditer(test)
    >>> print(f"输出：{b}")
    >>> for i in b:
            print(f"输出：{i}")
    
    
    输出：<callable_iterator object at 0x036E7BD0>
    输出：<re.Match object; span=(4, 9), match='2 two'>
    输出：<re.Match object; span=(10, 17), match='3 three'>
    

###  区别

  * 二者最大的区别在于一个返回列表，一个返回迭代器 

* * *

##  re.sub()与re.subn()

###  re.sub()

> re.sub(pattern, repl, string, count=0, flags=0)
>
>   * repl : 替换的字符串，也可为一个函数。
>   * count : 模式匹配后替换的最大次数，默认 0 表示替换所有的匹配。
>   * 最后返回替换结果
>

    
    
    >>> test = '1 one 2 two 3 three'
    >>> a=re.sub(r'(\d+)','xxx',test)
    >>> print(f"输出：{a}")
    >>> print(f"输出：{test}")
    
    
    输出：xxx one xxx two xxx three
    输出：1 one 2 two 3 three
    

###  re.subn()

> re.subn(pattern, repl, string, count=0, flags=0)  
>  参数含义同上

  * 功能与 ` re.subn ` 相同，但是返回一个元组 (字符串, 替换次数) 

    
    
    >>> test = '1 one 2 two 3 three'
    >>> a=re.subn(r'(\d+)','xxx',test)
    >>> print(f"输出：{a}")
    >>> print(f"输出：{test}")
    
    
    输出：('xxx one xxx two xxx three', 3)
    输出：1 one 2 two 3 three
    

##  re.split()

> re.split(pattern, string, maxsplit=0, flags=0)
>
> maxsplit：表示分割次数，默认为0，表示无限制

  * 用 pattern 分开 string 
  * 如果在 pattern 中捕获到括号，那么所有的组里的文字也会包含在列表里 

    
    
    >>> test = '1 one 2 two 3 three'
    >>> a = re.split(r'\d+', test)
    >>> b = re.split(r'(\d+)', test)
    >>> print(f"输出：{a}")
    >>> print(f"输出：{b}")
    
    
    输出：['', ' one ', ' two ', ' three']
    输出：['', '1', ' one ', '2', ' two ', '3', ' three']
    

##  正则表达式修饰符(匹配模式)

    
    
        re.I	使匹配对大小写不敏感
        re.L	做本地化识别匹配
        re.M	多行匹配，影响 ^ 和 $
                遇到\n视为新的一行，重新匹配 ^ 和 $
        re.S	使 . 匹配包括换行在内的所有字符
        re.U	根据Unicode字符集解析字符。这个标志影响 \w, \W, \b, \B.
        re.X	该标志通过给予你更灵活的格式以便你将正则表达式写得更易于理解。

