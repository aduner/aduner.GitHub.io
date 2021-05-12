---
title: Java并发/多线程-CAS原理分析
date: 2021-01-19 01:18
categories: ['Java']
tags: ['CAS', '并发编程', '多线程', 'Java']
---
#  什么是CAS

CAS 即 compare and swap，比较并交换。

CAS是一种原子操作，同时 CAS 使用乐观锁机制。

J.U.C中的很多功能都是建立在 CAS 之上，各种原子类，其底层都用 CAS来实现原子操作。用来解决并发时的安全问题。

#  并发安全问题

##  举一个典型的例子 ` i++ `

    
    
    public class AddTest {
      public volatile int i;
      public void add() {
        i++;
      }
    }
    

通过 ` javap -c AddTest ` 可以看到add 方法的字节码指令：

    
    
    public void add();
        Code:
           0: aload_0
           1: dup
           2: getfield      #2                  // Field i:I
           5: iconst_1
           6: iadd
           7: putfield      #2                  // Field i:I
          10: return
    

` i++ ` 被拆分成了多个指令：

  1. 执行 ` getfield ` 拿到原始内存值； 
  2. 执行 ` iadd ` 进行加 1 操作； 
  3. 执行 ` putfield ` 写把累加后的值写回内存。 

**假设一种情况：**

  * 当 ` 线程 1 ` 执行到 ` iadd ` 时，由于还没有执行 ` putfield ` ，这时候并不会刷新主内存区中的值。 
  * 此时 ` 线程 2 ` 进入开始运行，刚刚将主内存区的值拷贝到私有内存区。 
  * ` 线程 1 ` 正好执行 ` putfield ` ，更新主内存区的值，那么此时 ` 线程 2 ` 的副本就是旧的了。错误就出现了。 

##  如何解决？

最简单的，在 add 方法加上 synchronized 。

    
    
    public class AddTest {
      public volatile int i;
      public synchronized void add() {
        i++;
      }
    }
    

虽然简单，并且解决了问题，但是性能表现并不好。

最优的解法应该是使用JDK自带的 **CAS** 方案，如上例子，使用 ` AtomicInteger ` 类

    
    
    public class AddIntTest {
      public AtomicInteger i;
      public void add() {
        i.getAndIncrement();
      }
    }
    

##  底层原理

**CAS 的原理并不复杂：**

  * **三个参数，一个当前内存值 V、预期值 A、更新值 B**
  * **当且仅当预期值 A 和内存值 V 相同时，将内存值修改为 B 并返回 true**
  * **否则什么都不做，并返回 false**

拿 ` AtomicInteger ` 类分析，先来看看源码：

> 我这里的环境是Java11，如果是Java8这里一些内部的一些命名有些许不同。
    
    
    public class AtomicInteger extends Number implements java.io.Serializable {
        private static final long serialVersionUID = 6214790243416807050L;
    
        /*
         * This class intended to be implemented using VarHandles, but there
         * are unresolved cyclic startup dependencies.
         */
        private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
        private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");
    
        private volatile int value;
      
    		//...
    }
    

` Unsafe ` 类，该类对一般开发而言，少有用到。

` Unsafe ` 类底层是用 C/C++ 实现的，所以它的方式都是被 native 关键字修饰过的。

它可以提供硬件级别的原子操作，如获取某个属性在内存中的位置、修改对象的字段值。

**关键点：**

  * ` AtomicInteger ` 类存储的值在 ` value ` 字段中，而 ` value ` 字段被 ` volatile `

  * 在静态代码块中，并且获取了 ` Unsafe ` 实例，获取了 ` value ` 字段在内存中的偏移量 ` VALUE `

接下回到刚刚的例子：

如上， ` getAndIncrement() ` 方法底层利用 CAS 技术保证了并发安全。

    
    
    public final int getAndIncrement() {
      return U.getAndAddInt(this, VALUE, 1);
    }
    

` getAndAddInt() ` 方法：

    
    
    public final int getAndAddInt(Object o, long offset, int delta) {
      int v;
      do {
        v = getIntVolatile(o, offset);
      } while (!weakCompareAndSetInt(o, offset, v, v + delta));
      return v;
    }
    

` v ` 通过 ` getIntVolatile(o, offset) ` 方法获取，其目的是获取 ` o ` 在 ` offset ` 偏移量的值，其中
` o ` 就是 ` AtomicInteger ` 类存储的值，即 ` value ` ， ` offset ` 内存偏移量的值，即 ` VALUE `
。

**重点** ， ` weakCompareAndSetInt ` 就是实现 **CAS 的核心方法**

  * 如果 ` o ` 和 ` v ` 相等，就证明没有其他线程改变过这个变量，那么就把 ` v ` 值更新为 ` v + delta ` ，其中 ` delta ` 是更新的增量值。 
  * 反之 CAS 就一直采用自旋的方式继续进行操作，这一步也是一个原子操作。 

**分析：**

  * 设定 ` AtomicInteger ` 的原始值为 A， ` 线程 1 ` 和 ` 线程 2 ` 各自持有一份副本，值都是 A。 

  1. ` 线程 1 ` 通过 ` getIntVolatile(o, offset) ` 拿到 value 值 A，这时 ` 线程 1 ` 被挂起。 
  2. ` 线程 2 ` 也通过 ` getIntVolatile(o, offset) ` 方法获取到 value 值 A，并执行 ` weakCompareAndSetInt ` 方法比较内存值也为 A，成功修改内存值为 B。 
  3. 这时 ` 线程 1 ` 恢复执行 ` weakCompareAndSetInt ` 方法比较，发现自己手里的值 A 和内存的值 B 不一致，说明该值已经被其它线程提前修改过了。 
  4. ` 线程 1 ` 重新执行 ` getIntVolatile(o, offset) ` 再次获取 value 值，因为变量 value 被 volatile 修饰，具有可见性，线程A继续执行 ` weakCompareAndSetInt ` 进行比较替换，直到成功 

#  CAS需要注意的问题

##  使用限制

CAS是由CPU支持的原子操作，其原子性是在硬件层面进行保证的，在Java中普通用户无法直接使用，只能借助 ` atomic `
包下的原子类使用，灵活性受限。

但是CAS只能保证单个变量操作的原子性，当涉及到多个变量时，CAS无能为力。

原子性也不一定能保证线程安全，如在Java中需要与 ` volatile ` 配合来保证线程安全。

##  ABA 问题

###  概念

CAS 有一个问题，举例子如下：

  * ` 线程 1 ` 从内存位置 V 取出 A 
  * 这时候 ` 线程 2 ` 也从内存位置 V 取出 A 
  * 此时 ` 线程 1 ` 处于挂起状态， ` 线程 2 ` 将位置 V 的值改成 B，最后再改成 A 
  * 这时候 ` 线程 1 ` 再执行，发现位置 V 的值没有变化，符合期望继续执行。 

此时虽然 ` 线程 1 ` 还是成功了，但是这并不符合我们真实的期望，等于 ` 线程 2 ` **狸猫换太子** 把 ` 线程 1 ` 耍了。

**这就是所谓的ABA问题**

####  解决方案

> 引入原子引用，带版本号的原子操作。

把我们的每一次操作都带上一个版本号，这样就可以避免ABA问题的发生。既乐观锁的思想。

  * 内存中的值每发生一次变化，版本号都更新。 

  * 在进行CAS操作时，比较内存中的值的同时，也会比较版本号，只有当二者都没有变化时，才能执行成功。 

  * Java中的 ` AtomicStampedReference ` 类便是使用版本号来解决ABA问题的。 

##  高竞争下的开销问题

  * 在并发冲突概率大的高竞争环境下，如果CAS一直失败，会一直重试，CPU开销较大。 

  * 针对这个问题的一个思路是引入退出机制，如重试次数超过一定阈值后失败退出。 

  * 更重要的是避免在高竞争环境下使用乐观锁。 

* * *

