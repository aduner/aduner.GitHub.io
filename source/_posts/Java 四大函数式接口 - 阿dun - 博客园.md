---
title: Java 四大函数式接口
date: 2021-01-17 20:03
categories: ['Java']
tags: ['Java']
---
#  Java四大函数式接口

> 函数式接口： 只有一个方法的接口

##  ` Consumer<T> ` 消费型

给定一个参数，没有返回值

> void accept(T t);
    
    
    Consumer<String> c = (x) -> System.out.println("Hello World");
    

##  ` Supplier<T> ` 提供型

没有输入的参数，返回一个类型

> T get();
    
    
    Supplier<Integer> su = () -> 12;
    

##  ` Function<T,R> ` 函数型

给定一个参数，并返回一个值

> R apply(T t);
    
    
    Function<String, String> fun = (x) -> x.toUpperCase();
    

##  ` Predicate<T> ` 断定型

给定一个输入参数，返回一个 **布尔值**

> booolean test(T t);
    
    
    Predicate<String> pre = (str) -> str.length()>2;
    

