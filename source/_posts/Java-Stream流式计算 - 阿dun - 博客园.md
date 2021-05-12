---
title: Java-Stream流式计算
date: 2021-01-18 17:02
categories: ['Java']
tags: ['Java']
---
#  什么是 Stream流式计算

在 Java8 之前，如果我们想重新排序合并数据，一般是通过 for 循环或者 Iterator 迭代等方式进行操作。

但是这两种方式通常在数据量比较大的情况下，效率比较低。

在Java8中，添加了一个 **新的接口Stream** ，可以通过 **Lambda 表达式**
对集合进行各种非常便利、高效的聚合操作（Aggregate Operation），或者大批量数据操作 (Bulk Data Operation)。

**Stream流式计算**
：将要处理的元素看做一种流，流在管道中传输，并且可以在管道的节点上处理，包括过滤筛选、去重、排序、聚合等，并把结果发送到下一计算节点。

#  举例：对5个用户进行筛选

  * ID 必须是偶数 
  * 年龄必须大于23岁 
  * 用户名转为大写字母 
  * 用户名字母倒着排序 
  * 只输出一个用户！ 

**User类**

    
    
    class User {
        int id, age;
        String name;
        public User() {}
        public User(int id, String name, int age) {
            this.id = id;
            this.name = name;
            this.age = age;
        }
        public int getId() {
            return id;
        }
        public int getAge() {
            return age;
        }
        public String getName() {
            return name;
        }
    }
    

初始化的5个用户

    
    
    User u1 = new User(1,"a",21);
    User u2 = new User(2,"b",22);
    User u3 = new User(3,"c",23);
    User u4 = new User(4,"d",24);
    User u5 = new User(6,"e",25);
    

**使用传统的方式来实现**

    
    
    List<User> list = Arrays.asList(u1, u2, u3, u4, u5);
    List<String> nameList = new ArrayList<String>();
    for (User u : list) {
      if (u.getId() % 2 == 0)
        if (u.getAge() > 23)
          nameList.add(u.getName().toUpperCase());
    }
    nameList.sort(Comparator.reverseOrder());
    System.out.println(nameList.get(0));
    

**Stream进行实现**

    
    
    List<User> list = Arrays.asList(u1, u2, u3, u4, u5);
    list.stream()
      .filter(u -> {
        return u.getId() % 2 == 0;
      })
      .filter(u -> {
        return u.getAge() > 23;
      })
      .map(u -> {
        return u.getName().toUpperCase();
      })
      .sorted((uu1, uu2) -> {
        return uu2.compareTo(uu1);
      })
      .limit(1)
      .forEach(System.out::println);
    

#  Stream 操作分类

**Stream 的操作分为两大类：**

  * **中间操作（Intermediate operations）**

    * 只对操作进行了记录，即只会返回一个流，不会进行计算操作 
    * **无状态** （Stateless）：元素的处理不受之前元素的影响 
    * **有状态** （Stateful）：只有拿到所有元素之后才能继续下去 
  * **终结操作（Terminal operations）**

    * 终结操作是实现了计算操作 
    * **短路** （Short-circuiting）：遇到某些符合条件的元素就可以得到最终结果 
    * **非短路** （Unshort-circuiting）：必须处理完所有元素才能得到最终结果 

#  串行处理和并行处理

  * **串行处理操作** ： 

    * Stream 在执行每一步中间操作时，并不会做实际的数据操作处理，而是将这些中间操作串联起来，生成一个数据处理链表，通过 Spliterator 迭代器进行数据处理。 
    * 每执行一次迭代，就对所有的无状态的中间操作进行数据处理，而对有状态的中间操作，就需要迭代处理完所有的数据，再进行处理操作 
    * 最后进行终结操作的数据处理。 
  * **并行处理操作** （parallel()）： 

    * Stream 对中间操作基本跟串行处理方式是一样的 
    * 终结操作中，Stream 将结合ForkJoin对集合进行切片处理，ForkJoin 框架将每个切片的处理结果 Join 合并起来 

#  建议

在循环迭代次数较少的情况下，常规的迭代方式性能其实更好，尤其是服务器CPU只有单核

服务器是多核 CPU 的情况下，Stream 的并行迭代优势明显。

所以在平时处理大数据的集合时，应该尽量考虑将应用部署在多核 CPU 环境下，并且使用 Stream 的并行迭代方式进行处理

