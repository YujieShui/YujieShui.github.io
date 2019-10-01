---
title: 【Java并发编程】Java线程的生命周期
toc: true
categories:
- Java
- Java并发编程
tags:
  - java
  - 并发编程
abbrlink: 541ca933
date: 2019-09-08 10:47:53
---

在 Java 领域，实现并发程序的主要手段是多线程。线程是操作系统的概念，虽然不同语言对线程操作进行了不同的封装，但是万变不离其综。

通用的线程模型可以用「五态模型」来描述，它们分别是：初始状态，可运行状态，运行状态，休眠状态和终止状态。

![通用线程转换图——五态模型](http://image.shuiyujie.com/2019-09-08-12-04-35.png)

这五种状态在不同编程语言里会有简化合并。例如，C 语言的 POSIX Threads 规范，就把初始状态和可运行状态合并了；Java 语言里则把可运行状态和运行状态合并了，这两个状态在操作系统调度层面有用，而 JVM 层面不关心这两个状态，因为 JVM 把线程调度交给操作系统处理了。

除了简化合并，这五种状态也有可能被细化，比如，Java 语言里就细化了休眠状态。

<!-- more -->

# Java 线程的生命周期

Java 语言中的线程有 6 种状态

- NEW (初始状态)
- RUNNABLE (可运行/运行状态)
- BLOCKED (阻塞状态)
- WATTING (无限期等待)
- TIMED_WAITING (有时限等待)
- TERMINATED (终止状态)

其中 BLOCKED、WATTING 和 TIMED_WAITING 在操作系统层面同属于前面提高的休眠状态。简化的 Java 的生命周期图，如下所示：

![Java中的线程状态转换图](http://image.shuiyujie.com/2019-09-08-12-11-20.png)

那么些状态是如何产生，如何进行转换的呢？

# 从 NEW 到 RUNNABLE

NEW 状态指的是**在编程语言层面创建了线程，但是操作系统层面还未创建线程。**Java 中刚创建出来的 Thread 对象就是 NEW 状态，创建 Thread 对象的方法有两种：

1. 继承 Thread 对象，重写 run() 方法
2. 实现 RUNNABLE 接口，重写 run() 方法

此时通过`MyThread myThread = new MyThread()`或者`Thread thread = new Thread(new Runner())`创建出来的线程就处在 NEW 状态。

如果想要将线程转换到 RUNNABLE 状态，只需要使用`start()`方法，即`myThread.start()` 或者 `thread.start()`。	

# 从 RUNNABLE 到 BLOCKED

BLOCK 表示阻塞状态，在 Java 中只有一种情况会触发 RUNNABLE 到 BLOCKED 的转换，那就是**线程等待 synchronized 锁**。synchronized 修饰的代码块同一时刻只允许一个线程执行，其他线程就只能等待。当等待的线程拿到 synchronized 隐式锁的时，就会从 BLOCKED 切换到 RUNNABLE。

这里的 BLOCKED 和操作系统层面的阻塞有所区别。在操作系统层面，当线程调用阻塞时 API，比如说进行 IO 操作，那么该线程就会处于休眠状态。但是在 JVM 层面 Java 程序的状态并不会发生改变，任然会保持为了 RUNNABLE 状态。**JVM 层面并不关系操作系统调度相关的状态**。在 JVM 看来，等待 CPU 的使用权，与等待 I/O 没有区别，都是在等待某个资源，所以都归入到 RUNNABLE 状态。

# 从 RUNNBALE 到 WAITING

总体来说，有三种场景会触发这种转换。

- 第一种场景，获得 synchronized 隐式锁的线程，调用无参数的 Object.wait() 方法。
- 第二种场景，调用无参数的 Thread.join() 方法。其中的 join() 是一种线程同步方法，例如有一个线程对象 thread A，当调用 A.join() 的时候，执行这条语句的线程会等待 thread A 执行完， 而等待中的这个线程，其状态会从 RUNNABLE 转换到 WAITING。当线程 thread A 执行完，原来等待它的线程又会从 WAITING 状态转换到 RUNNABLE。
- 第三种场景，调用 LockSupport.park() 方法。其中的 LockSupport 对象，也许你有点陌生，其实 Java 并发包中的锁，都是基于它实现的。调用 LockSupport.park() 方法，当前线程会阻塞， 线程的状态会从 RUNNABLE 转换到 WAITING。调用 LockSupport.unpark(Thread thread) 可 唤醒目标线程，目标线程的状态又会从 WAITING 状态转换到 RUNNABLE。

# 从 RUNNBALE 到 TIMED_WATING 

有五种场景会触发这种转换：

1. 调用带超时参数的 Thread.sleep(long millis) 方法；

2. 获得 synchronized 隐式锁的线程，调用带超时参数的 Object.wait(long timeout) 方法；

3. 调用带超时参数的 Thread.join(long millis) 方法；

4. 调用带超时参数的 LockSupport.parkNanos(Object blocker, long deadline) 方法；

5. 调用带超时参数的 LockSupport.parkUntil(long deadline) 方法。

# 从 RUNNBALE 到 TERMINATED

线程执行完 run() 方法后，会自动转换到 TERMINATED 状态，当然如果执行 run() 方法的时候异常抛出，也会导致线程终止。有时候我们需要强制中断 run() 方法的执行，例如 run() 方法访 问一个很慢的网络，我们等不下去了，想终止怎么办呢？Java 的 Thread 类里面倒是有个 stop() 方法，不过已经标记为 @Deprecated，所以不建议使用了。正确的姿势其实是**调用 interrupt() 方法**。

interrupt() 会通过异常或者主动监测的方式通知线程，线程有机会执行后续操作，也可以无视这个通知。