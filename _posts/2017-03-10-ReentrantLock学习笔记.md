---
title: ReentrantLock
categories:
 - Java
tags: Java
---

在java多线程中可以使用synchronized关键字来实现线程之间同步互斥，
在jdk1.5中新增加了ReentrantLock类，能够达到同样的效果，还扩展了更强大的功能。
比如说**嗅探锁定**，**多路分支通知**。

重点说明下多路分支通知。

它的基本原来也就是之前的wait,notify,notifyAll方法。在此基础上进行了选择性通知。
接下来我以伪代码做说明：
```
    private ReentrantLock lock = new ReentrantLock();
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    
    void test1() {
        lock.lock();
        //do sth
        c1.await();
    }
    
    void test2() {
        lock.lock();
        //do sth
        c2.await();
    }
    
    void test3() {
        lock.lock();
        //do sth
        c1.signalAll();
    }
    
    void test4() {
        lock.lock();
        //do sth
        c2.signal();
    }
```

可根据我们的需求来决定线程执行的先后顺序。

但是同一时间只有一个线程在执行ReentrantLock.lock()方法后面的任务，这样虽然保证了实例变量的线程安全性，
但是效率却非常低下。

为了解决这个问题，更推荐使用ReentrantReadWriteLock类。
读写锁有两个锁，读锁又成为共享锁，写锁又称为排他锁。
也就是说我们可以异步的进行多个读操作，但是写操作只能有一个线程在进行。

ReentrantLock获取锁的三种方式：lock(),tryLock(),tryLock(long timeout,TimeUnit unit)
它与Synchronized有着相同的并发性和内存语义。
区别在于，A获取对象锁后，B等待A释放A锁。Synchronized:A如果不释放，B将一直等待下去，不能被中断。
而ReentrantLock,如果A不释放，可以使B在等待足够长时间后中断等待，干别的事情。
