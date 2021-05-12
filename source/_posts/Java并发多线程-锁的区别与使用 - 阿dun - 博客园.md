---
title: Java并发/多线程-锁的区别与使用
date: 2021-01-15 20:59
categories: ['Java']
tags: ['Java', '多线程']
---
#  锁类型

###  可中断锁

  * 在等待获取锁过程中可中断 
  * Lock就是可中断锁 

###  公平锁/非公平锁

  * 公平锁是指多个线程按照申请锁的顺序来获取锁。 
  * 非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。 
  * 对于ReentrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。 
  * 对于Synchronized而言，也是一种非公平锁。由于其并不像ReentrantLock是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。 

###  可重入锁

  * 可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。 
  * 对于Java ReentrantLock而言, 他的名字就可以看出是一个可重入锁，其名字是Re entrant Lock重新进入锁。 
  * 对于Synchronized而言,也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。 

    
    
    synchronized void setA() throws Exception{
     
        Thread.sleep(1000);
     
        setB();
     
    }
     
    synchronized void setB() throws Exception{
     
        Thread.sleep(1000);
     
    }
    

###  独享锁/共享锁

  * 独享锁是指该锁一次只能被一个线程所持有。 
  * 共享锁是指该锁可被多个线程所持有。 
  * 对于Java ReentrantLock而言，其是独享锁。但是对于Lock的另一个实现类ReadWriteLock，其读锁是共享锁，其写锁是独享锁。 
  * 读锁的共享锁可保证并发读是非常高效的，读写，写读 ，写写的过程是互斥的。 
  * 独享锁与共享锁也是通过AQS来实现的，通过实现不同的方法，来实现独享或者共享。 
  * Synchronized是独享锁。 

###  互斥锁/读写锁

  * 上面讲的独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。 
  * 互斥锁在Java中的具体实现就是ReentrantLock 
  * 读写锁在Java中的具体实现就是ReadWriteLock 

###  乐观锁/悲观锁

  * 乐观锁与悲观锁不是指具体的什么类型的锁，而是指看待并发同步的角度。 
  * 悲观锁认为对于同一个数据的并发操作，一定是会发生修改的，哪怕没有修改，也会认为修改。因此对于同一个数据的并发操作，悲观锁采取加锁的形式。悲观的认为，不加锁的并发操作一定会出问题。 
  * 乐观锁则认为对于同一个数据的并发操作，是不会发生修改的。在更新数据的时候，会采用尝试更新，不断重新的方式更新数据。乐观的认为，不加锁的并发操作是没有事情的。 
  * 从上面的描述我们可以看出，悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升。 
  * 悲观锁在Java中的使用，就是利用各种锁。 
  * 乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。 

###  分段锁

  * 分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。 
  * 分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。 

###  偏向锁/轻量级锁/重量级锁

  * 这三种锁是指锁的状态，并且是针对Synchronized。在Java 5通过引入锁升级的机制来实现高效Synchronized。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。 
  * 偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。 
  * 轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。 
  * 重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让其他申请的线程进入阻塞，性能降低。 

###  自旋锁

  * 自旋锁原理非常简单，如果持有锁的线程能在很短时间内释放锁资源，那么那些等待竞争锁的线程就不需要做内核态和用户态之间的切换进入阻塞挂起状态，它们只需要等一等（自旋），等持有锁的线程释放锁后即可立即获取锁，这样就 **避免用户线程和内核的切换的消耗** 。 
  * 但是线程自旋是需要消耗cup的，说白了就是让cup在做无用功，如果一直获取不到锁，那线程也不能一直占用cup自旋做无用功，所以需要设定一个自旋等待的最大时间。 
  * 如果持有锁的线程执行的时间超过自旋等待的最大时间扔没有释放锁，就会导致其它争用锁的线程在最大等待时间内还是获取不到锁，这时争用线程会停止自旋进入阻塞状态。 

#  Synchronized与Static Synchronized

区别

  * static synchronized方法是类锁，synchronized方法是对象锁 

  * synchronized相当于 this.synchronized，static synchronized相当于Something.synchronized 

###  举例

    
    
    pulbic class Test(){
        public synchronized void A(){}
        public synchronized void B(){}
        public static synchronized void cA(){}
        public static synchronized void cB(){}
    }
    

新建两个实例，x和y

    
    
    1 x.A()与x.B() 
    2 x.A()与y.A()
    3 x.cA()与y.cB()
    4 x.A()与Test.cA()
    

四种情况哪组方法何以被多个以上线程同时访问？

  * 情况1：同一个实例上锁肯定不行 
  * 情况2：是针对不同实例的，因此可以同时被访问 
  * 情况3：因为static synchronized为类锁，所以不同实例之间仍然会被限制 
  * 情况4：可以，类锁和实例锁互不干扰 

#  Lock

##  定义

Lock是一个接口

    
    
    public interface Lock {
        void lock();
        void lockInterruptibly() throws InterruptedException;
        boolean tryLock();
        boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
        void unlock();
        Condition newCondition();
    }
    

  * ` lock() ` 、 ` tryLock() ` 、 ` tryLock(long time, TimeUnit unit) ` 和 ` lockInterruptibly() ` 是用来获取锁的。 
  * ` unLock() ` 方法是用来释放锁的。 

##  四种获取Lock的方法区别

###  ` lock() `

平常使用得最多的一个方法，就是用来获取锁。如果锁已被其他线程获取，则进行等待。

使用Lock必须在 ` try{}catch{} ` 块中进行，并且将释放锁的操作放在 ` finally `
块中进行，以保证锁一定被被释放，防止死锁的发生。

    
    
    Lock lock = ...;
    lock.lock();
    try{
        //处理任务
    }catch(Exception ex){
         
    }finally{
        lock.unlock();   //释放锁
    }
    

###  ` tryLock() `

` tryLock() ` 有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果锁已被其他线程获取，则返回false。

这个方法会立即返回，在拿不到锁时也不会一直在那等待。

    
    
    Lock lock = ...;
    if(lock.tryLock()) {
         try{
             //处理任务
         }catch(Exception ex){
             
         }finally{
             lock.unlock();   //释放锁
         } 
    }else {
        //如果不能获取锁，则直接做其他事情
    }
    

###  ` tryLock(long time, TimeUnit unit) `

此方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

` lockInterruptibly() `
方法比较特殊，当通过这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。也就使说，当两个线程同时通过 `
lock.lockInterruptibly() `
想获取某个锁时，假若此时线程A获取到了锁，而线程B还在等待，那么对线程B调用threadB.interrupt()方法中断线程B的等待过程。

###  ` lockInterruptibly() `

当通过lockInterruptibly()方法获取某个锁时，如果不能获取到，只有进行等待的情况下，是可以响应中断的。

而用synchronized修饰的话，当一个线程处于等待某个锁的状态，是无法被中断的，只有一直等待下去。

当一个线程获取了锁之后，是不会被interrupt()方法中断的。单独调用interrupt()方法不能中断正在运行过程中的线程，只能中断阻塞过程中的线程。

` lockInterruptibly() ` 的声明中抛出了异常，所以 ` lock.lockInterruptibly() `
必须放在try块中或者在调用 ` lockInterruptibly() ` 的方法外声明抛出InterruptedException。

    
    
    public void method() throws InterruptedException {
        lock.lockInterruptibly();
        try {  
         //.....
        }
        finally {
            lock.unlock();
        }  
    }
    

#  synchronized与Lock的区别

类别  |  synchronized  |  Lock  
---|---|---  
存在层次  |  Java的关键字，在jvm层面上  |  是一个接口  
锁的释放  |  以获取锁的线程执行完同步代码，释放锁  
线程执行发生异常，jvm会让线程释放锁  |  在finally中必须释放锁，不然容易造成线程死锁  
锁的获取  |  死等  |  有多个锁获取的方式，可以尝试获得锁，线程可以不用死等  
锁状态  |  无法判断  |  可以判断  
锁类型  |  可重入  
不可中断  
非公平  |  可重入 可判断 可公平（两者皆可）  
性能  |  少量同步时更好  |  大量同步时更好  
  
###  synchronized和lock的用法区别

synchronized：在需要同步的对象中加入此控制，synchronized可以 **加在方法上** ，也可以加在特定代码块中，括号中表示需要锁的对象。

    
    
    public synchronized void Test(){}
    
    synchronized(temp){
    	// 操作
    }
    

Lock：需要显示指定起始位置和终止位置。前文已经有示例这里不再举例。

###  synchronized和Lock性能区别

  * 大量同步时Lock可以提高多个线程进行读操作的效率。（可以通过readwritelock实现读写分离） 
  * 在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态 
  * ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock还能维持常态。 

###  synchronized和Lock用途区别

两者在一般情况下没有什么区别，但在非常复杂的同步应用中，应该使用Lock，例如：

  * 某个线程在等待一个锁的控制权的这段时间需要中断 

  * 需要分开处理一些wait/notify，JUC里面的Condition，能够实现精准唤醒 

  * 需要公平锁功能 

