---
title: JavaScript 浅析数组对象与类数组对象
date: 2020-01-22 21:17
categories: ['前端']
tags: ['数组', 'js', 'JavaScript', '前端', '类数组']
---
#  数组(Array对象)

> 数组就是一组数据.  
>  
>  在JavaScript中没有数组这种数据类型.数组时对象创建的.

>   * 键(下标): 用于区分数组中不同数值的标志就是键,初始键为0.
>     * 以数字作为下标的键,这种数组称之为索引数组.
>     * 以字符串作为下标的键,这种数组称之为关联数组
>     * **注意:** 在JavaScript中只有索引数组,y没有关联数组,必须是从0开始连续的索引数组!
>   * 值:每个键对应的数据就是值.
>   * 键值对:键+值 就是键值对
>

##  数组的操作

###  创建数组方法

  * new Array(); 

    * var 变量 = new Array() ; 创建一个空数组 
    * var 变量 = new Array(值,值...) ; 创建具有指定元素的数组 
    * var 变量 = new Array(整数); 创建具有指定长度的数组 
  * 使用对象字面量 

    * var 变量 = []; 创建一个空数组 
    * var 变量 = [值,值...];创建具有指定元素的数组 
    * var 变量 = [整数] ; 创建一个指定整数元素的数组,并不是长度!~ 

###  添加与修改数组元素

  * 数组变量[数组变量.length] = 值; 

> a[a.length] = 1;//添加

  * 数组变量[指定下标] = 新值; 

> a[0] = 1;修改

  * 数组变量[超过当前数组长度的数值] = 值; 

> a = [1,2,3];  
>  
>  a[5] = 5;  
>  
>  输出：[1, 2, 3, empty × 2, 5]

###  删除数组元素

  * delete 数组变量[指定下标]; 

> deldet a[0];

  * **注意:** 这种方式仅仅可以删除数组的元素值，不可以删除键，删除操作之后，数组的长度保持不变。如果想彻底删除元素需要使用splice这个方法。 

###  使用数组元素

  * 数组变量[指定下标] ; 

###  遍历数组元素

  * 第一种 
    
        var x = [1,2,3];
    for (var i=0;i<x.length;i++){
        console.log(x[i]);
    }
    

  * 第二种 
    
        var x = [1,2,3];
    for (var i of x){
        console.log(i);
    }
    

输出均为以下

    
        1
    2
    3
    

##  多维数组

> 通俗解释如下

  * 一维数组: 

> 如果一个元素中的所有值都不是数组,那么这个数组就是一个一维数组.

  * 二维数组: 

> 如果一个一维数组的元素中包含一维数组,那么这个数组就是二维数组

  * 三维数组: 

> 如果一个一维数组的元素中包含二维数组,那么这个数组就是三维数组

  * 多维数组: 

> 如果一个数组的维度超过三维,统称为多维数组

  * 多维数组的访问操作: 

> 数组变量[键][键]...

##  数组相关的函数

###  concat()

  * 连接数组元素 
  * **格式:**

> 新变量 = 数组变量.concat(值,值...)  
>  
>  var b = a.concat(a, 1, "x");

  * 参数值,可以是正常数值也可以是数组类型，都可以连接。 
  * 给数组就在后面添加所有数组元素，给普通数值就添加普通数值 

###  join()

  * 使用字符将数组元素连接成一个字符串,默认为 ` , `
  * **格式:**

> 新变量 = 数组变量.join(制定字符串)  
>  
>  var b = a.join("");// ` "" ` 空字符就等于没有连接字符串

###  pop()

  * 在数组的结尾处弹出一个元素(直接改变原有数组) 
  * **格式:**

> 结果变量 = 数组变量.pop();  
>  
>  var b = a.pop();

  * 注意:该方法直接操作原有数组.返回值为弹出的元素 

###  push()

  * 在数组的结尾处添加元素(直接改变原有数组) 
  * **格式:**

> 结果变量 = 数组变量.push(值,值..)  
>  
>  var b = a.push("1", "2");

  * 注意:该方法直接操作原有数组.返回值为添加后的数组长度 

###  shift()

  * 在数组的开头移除一个元素(直接改变原有数组) 
  * **格式:**

> 结果变量 = 数组变量.shift();  
>  
>  var b = a.shift();

###  unshift()

  * 在数组的开头添加元素(直接改变原有数组) 
  * **格式:**

> 结果变量 = 数组变量.unshift(值,值...)  
>  
>  var b = a.unshift("1", "2");  
>  ​

###  reverse()

  * 数组倒转方法 
  * **格式:**

> 结果变量 = 数组变量.reverse();  
>  
>  var b = a.reverse();

###  sort()

  * 数组排序方法 
  * 字符串排序,使用ascii进行排序操作. 
    * **格式:**

> 结果变量 = 数组变量.sort();  
>  
>  var b = a.sort();

  * 数字排序,使用数字大小排序 
    * **格式:**

> 结果变量 = 数组变量.sort(回调函数);  
>  
>  var b = a.sort(c);

    * 回调函数的格式要求: 
      * 必须传入2个参数. 
      * 必须返回正数,负数或者0; 

###  slice()

  * 切割数组,并且返回其中的一段 
  * **格式:**

> 结果变量 = 数组变量.slice(开始下标,结束下标);  
>  
>  var b = a.slice(1,3);

  * **注意:** 包含开始位置的元素,但是不包含结束位置的元素 
  * 其中参数为位置参数,正数表示从前向后数,负数从后向前数. 

###  splice()

  * 数组万能操作方法(直接改变原有数组) 
  * 在指定位置添加元素 

> 结果变量 = 数组变量.splice(制定位置,0,新增元素..)  
>  
>  var b = a.splice(1,0,"ABC");

  * 在制定位置删除元素 

> 结果变量 = 数组变量.splice(制定位置,删除个数);  
>  
>  var b = a.splice(1,2);

  * 在制定位置替换元素 

> 结果变量 = 数组变量 .splice(制定位置,删除个数,新增元素)  
>  
>  var b = a.splice(1,2,"ABC","DEF");

#  类数组对象

> 在JavaScript中有一种性质与数组相似特殊的对象，我们称之为类数组对象，最常见的便是 argumengs对象。

###  定义

>   * 可以通过索引访问元素，并且拥有 length 属性；
>   * 没有数组的其他方法，例如 push ， forEach ， indexOf 等。
>

###  举例

类数组

    
    
        var arrLike = {
            0: 'Java',
            1: 'Python',
            2: 'PHP',
            length: 3
        }
    

同款数组

    
    
    var arr = ["Java", "Python", "PHP"];
    

##  对比数组

  * 类数组对象在访问、赋值、获取长度上的操作与数组是一致的 

    * 访问 

> console.log(arr[0]); // Java  
>  
>  console.log(arrLike[0]); // Java

    * 赋值 

> arr[0] = 'JavaScript';  
>  
>  arrLike[0] = 'JavaScript';

    * 获取对象的长度 

> console.log(arr.length); // 3  
>  
>  console.log(arrLike.length); // 3

  * 类数组对象与数组的区别是类数组对象不能直接使用数组的方法 

    * 类数组对象使用数组方法时会报错 

> arrLike.push("C++"); // Uncaught TypeError: arrLike.push is not a function

##  转换

> 有时候我们需要让类数组有数组的特性，这时候就需要转换，可以是直接转换也可以间接转换。

###  间接

> 通过 Function.call 或Function.apply 方法

  * Function.call 

    
    
    Array.prototype.push.call(arrLike, 'C++');
    console.log(arrLike);
     // { '0': 'Java', '1': 'Python', '2': 'PHP', '3': 'C++', length: 4 }
    

  * Function.apply 

    
    
    Array.prototype.push.apply(arrLike, ['C++']);
    console.log(arrLike);
     // { '0': 'Java', '1': 'Python', '2': 'PHP', '3': 'C++', length: 4 }
    

###  直接

  * Array.prototype.slice 将类数组转换为数组 

> Array.prototype.slice.call(arrLike,0);  
>  
>  Array.prototype.slice.apply(arrLike,0);

  * Array.prototype.splice 将数组转换为类数组 

> Array.prototype.splice.call(arrLike,0);  
>  
>  Array.prototype.splice.apply(arrLike,0);

* * *

