---
title: 【Java并发编程】Lock&Condition
toc: true
categories:
- Java
- Java并发编程
tags:
  - java
  - 并发编程
  - 管程
  - 并发工具类
abbrlink: 3e6f77c3
date: 2019-09-09 22:18:02
---

在并发编程中有两个核心问题:一个是**互斥**,即同一时刻只允许一个线程访问共享资源;另一个是**同步**,即线程之间如何通信和协作。解决这两个问题的方法前人已经替我们总结出来理论模型,分别是**管程模型**和**信号量模型**.

![MESA管程模型](http://image.shuiyujie.com/2019-09-09-22-20-56.png)

Java中实现管程有两种方式

1. 一是使用`synchronized`关键字给代码块添加隐式锁实现互斥,同时使用`notify()`和`notifyAll()`实现**同步**
2. 二是使用 Java SDK 并发包下的  Lock 和 Condition 两个接口,来实现管程.
   1. Lock 的特性包括:能够响应中断、支持超时和非阻塞地获取锁
   2. Condition 实现了管程模型里面的条件变量。

*注：Java 参考了 MESA 模型，语言内置的管程（synchronized）对 MESA 模型进行了精简。MESA 模型中，条件变量可以有多个，Java 语言内置的管程里只有一个条件变量。Java 并发包下的 Lock&Condition则则支持多个条件变量。*

<!-- more -->

# 为什么要用 lock&condition 再次实现管程

Java 已经通过`synchronized`实现了管程,那么为什么 jdk1.5 还要在并发包中用 lock&condition 来实现管程? 这两种实现方式存在什么区别么? 它们各自有什么样的应用场景?

在之前的[死锁问题](https://shuiyujie.com/post/3d624a82.html)一文中提到了死锁的产生必须同时具备 4 个条件,我们只需要破坏其中任意一个条件都能够预防死锁的发生.其中有一个条件是**不可抢占**,也就是线程 T1 持有某一个资源,其他线程不能强行抢占它持有的资源.

当我们使用`synchronized`加锁时,当它持有锁A,但是无法持有到锁 B, 时线程就会直接进入阻塞状态,此时一旦发生死锁就没有办法唤醒这个线程了.Lock&Condition的出现就是为了做到**破坏不可抢占条件**.

具体来说,怎么才能破坏不可抢占条件呢?有这样 3 个思路:

1. **能够响应中断**.当线程进入阻塞状态之后,我们能够发送一个中断信号给这个阻塞的线程.该线程能够响应中断信号,就有机会释放持有的锁 A,就能破坏不可抢占的条件.
2. **超时机制**.线程在一段时间内无法获取到锁,它不会进入阻塞状态,而是返回错误,那么这个线程也有机会释放持有的锁.
3. **非阻塞地获取锁**.如果尝试获取锁失败，并不进入阻塞状态，而是直接返回.

这三种方案可以弥补 synchronized 实现的管程无法破坏不可抢占条件的问题,具体来说  Lock 接口实现这里这样 3 个方法:

```java
// 支持中断的 API
void lockInterruptibly() throws InterruptedException;
// 支持超时的 API
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
// 支持非阻塞获取锁的 API
boolean tryLock();
```

# 用两个条件变量实现阻塞队列

 Java 语言内置的管程里只有一个条件变量，而 Lock&Condition 实现的管程是支持多个条件变量的，这是二者的一个重要区别。

在很多并发场景下，支持多个条件变量能够让我们的并发程序可读性更好，实现起来也更容易。例如，实现一个阻塞队列，就需要两个条件变量。

一个阻塞队列，需要两个条件变量，一个是队列不空（空队列不允许出队），另一个是队列不满（队列已满不允许入队）。

```java
public class BlockedQueue<T> {
    final Lock lock = new ReentrantLock();
    // 条件变量,队列不满
    final Condition notFull = lock.newCondition();
    // 条件变量.队列不空
    final  Condition notEmpty = lock.newCondition();

    // 入队
    void enq(T x) {
        lock.lock();
        try {
            while(队列已满){
                // 等待队列不满
                notFull.await();
            }
            // 省略入队操作
            // 入队后,通知可出队
            notEmpty.signal();
        }finally {
            lock.unlock();
        }
    }

    // 出队
    void deq(){
        lock.lock();
        try{
            while(队列已空){
                // 等待队列不空
                notEmpty.await();
            }
            // 省略出队操作
            // 出队后,通知可入队
            notFull.signal();
        }finally {
            lock.unlock();
        }
    }
}
```



