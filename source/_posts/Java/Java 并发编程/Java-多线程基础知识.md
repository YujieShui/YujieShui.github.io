---
title: Java 多线程基础知识
toc: true
categories:
  - Java
tags:
  - java
  - 并发编程
abbrlink: 1d67cce0
date: 2017-11-20 20:11:14
---

计算机发明之初，单个 CPU 在同一时间点只能执行单个任务，也就是单任务阶段。紧接着发展为多任务阶段，在单个 CPU 上能执行多个进程，但是此处的并行执行并不是指在同一时间执行多个任务，而是指由系统对进程进行调度轮流使用 CPU。

接着发展到现在的多线程阶段，一个程序内部能够运行多个线程，每个线程都可以被看做运行在一个 CPU 上，此时计算机真正做到了在同一时间点能够执行多个任务。

<!-- more -->

# 进程和线程

进程和线程怎么界定？一个进程包含着一个或多个线程。

当我们启动一个应用程序的时候，操作系统将加载这个应用程序并为当前程序**开辟一块独立的内存空间**，这片空间就专门用来负责这个程序的运行。这就是一个进程，我们可以把它想象成一个车间。

假设我们启动的程序是微信，我们可以使用微信和多个人聊天并且互不干扰。同时和不同的人聊天就是将进程划分成多个区域，每个区域就是一个线程。如果把进程比作车间，线程就像车间中的工人，工人们各司其职维持车间的正常运转。

# 多线程运行原理

多线程就是在一个进程中开启多个线程，让多个线程去实现不同功能。多线程最常见的就是 GUI 程序，比如说使用 IDE 运行程序的同时我们还能用 IDE 继续写代码，这就用到了多线程。

多线程运行则是利用 CPU 在线程中做时间片切换。CPU 负责运行程序且一次只能运行一个程序，**多线程的实现依靠 CPU 在线程间快速切换，**由于切换的时间很快就像同时运行多个线程一样。

多线程切换过程中，它需要先存储当前线程的本地的数据，程序指针等，然后载入另一个线程的本地数据，程序指针等，最后才开始执行。所以多线程一般能提高程序运行效率，但也不能无节制地开启线程。

# Java 实现多线程的两种方式

多线程的两种创建方式分别为继承 Thread 类以及实现 Runnable 接口。一般用实现 Runnable 接口的方式。

我们通过调用线程的 `start()` 方法新建一个线程并启动它，下面是两种主要实现方式的示例。

## 继承 Thread 类

```java
public class MyThread1 extends Thread {

    private int count = 5;

    @Override
    public void run() {
        for (int i = 5; i > 0 && count>0; i--,count--) {
            System.out.println("count----" + count);
        }
    }

    public static void main(String[] args) {

        new MyThread1().start();
        new MyThread1().start();
        new MyThread1().start(); 
    }
}
```

可以创建一个实例直接 `start()`，因为 Thread 类中定义了 `start()` 方法。

## 实现 Runnable 接口

```java
public class MyThread2 implements Runnable{

    private int count = 5;

    @Override
    public void run() {
        for (int i = 5; i > 0 && count>0; i--,count--) {
            System.out.println("count----" + count);
        }
    }

    public static void main(String[] args) {
        MyThread2 myThread2 = new MyThread2();
        new Thread(myThread2).start();
        new Thread(myThread2).start();
        new Thread(myThread2).start();
    }
}
```

但是他无法直接创建实例之后直接 `start()`,查看源码可以看到 Runnable 接口只有一个 `run()` 方法没有 `start()`方法，所以他必须由 Thread 来启动。

## 两者优缺点比较

### 实现 Runable() 避免由于 Java 的单继承特性而带来的局限

Java 使用 extends 关键字实现继承，可以理解成全盘接受了父类的特性。使用 implents 关键字实现接口，对类的功能进行拓展。**要注意的是 Java 的继承是单继承，也就是只能继承一个父类。但是每个类都能实现多个接口。**

当一个类需要实现多线程的时候，如果他已经继承了其他的父类则不能再继承 Thread 类，但是即使他实现了其他接口，他任然能实现 Runable 接口创建多线程。

但是当我们访问当前线程时需要使用`Thread.currentThread()`方法。

### 继承 Thread 类编写简单，方便理解

通过继承 Thread 类的方法实现多线程，调用当前类只需要使用 `this` 关键字即可进行访问。但是使用这种方式之后就不能继承其他父类。

**因此一般都是用实现 Runable() 接口的形式。本篇中使用显示创建线程的方式，而创建线程更好的方式是使用线程池。**

# 多线程 run() 和 start() 的区别

通过运行`run()`方法和`start()`方法，看看输出结果有什么区别。

## run() 方法

```java
public class MyThread3 extends Thread {

    private int count = 5;

    @Override
    public void run() {
        for (int i = 5; i > 0 && count>0; i--,count--) {
            System.out.println(this.currentThread().getName() + "----count----" + count);
        }
    }

    public static void main(String[] args) {

        MyThread3 myThread1 = new MyThread3();
        myThread1.setName("Thread1");
        myThread1.run();

        MyThread3 myThread2 = new MyThread3();
        myThread2.setName("Thread2");
        myThread2.run();

    }
}
```

输出结果为：

```
thread1----count----5
thread1----count----4
thread1----count----3
thread1----count----2
thread1----count----1
thread2----count----5
thread2----count----4
thread2----count----3
thread2----count----2
thread2----count----1
```

**Thread1 和 Thead2 只是在主线程上顺序执行，并没有开启新的线程。**

## start() 方法

如果将 `run()` 改成 `start()`输出结果为:

```
thread2---count----5
thread3---count----5
thread1---count----5
thread3---count----3
thread3---count----1
thread2---count----4
thread1---count----2
```

可以看到两个线程在抢占资源，成功创建了两个线程。

## 结果对比

创建了两个相同的线程对象，但是结果有明显的差异。`run()` 的运行结果始终是顺次递减，线程1执行完之后再执行线程2，表明他们是在主线程上顺序运行，并没有开启新的线程。

但是使用`start()`方法则可以看到线程1和线程2在竞争资源，说明成功开启了两个线程。

# 线程 start() 和 run() 的区别

**先看一下 Runnable 接口**

```java
public interface Runnable {
    public abstract void run();
}
```

其中只有一个 run() 方法，注释是这么写的：

> When an object implementing interface <code>Runnable</code> is used to create a thread

*所以用于实现了 run() 方法的对象可以被用于创建一个线程*

**再看 Thread 类**

```java
public class Thread implements Runnable {
    /* What will be run. */
    private Runnable target;
    
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```

Thread 实现了 Runable 的接口,可以传入一个 Runnable，执行 Runnable 的 run() 方法,他说:

> If this thread was constructed using a separate <code>Runnable</code> run object, then that <code>Runnable</code> object's <code>run</code> method is called; otherwise, this method does nothing and returns.

说明 Thread 类的作用就是用于运行 Runable 的实例，将实现了 Runable 方法的实例传入 Thread 类就可以运行该实例的`run()`方法

再看`start()` 方法的说明：

使用 `start()` 让线程会调用`run()`之后在虚拟机上执行

> Causes this thread to begin execution; the Java Virtual Machine calls the <code>run</code> method of this thread.

```java
public synchronized void start() {
        if (threadStatus != 0)
            throw new IllegalThreadStateException();
        
        /*把线程加到线程组中，表示这个线程将会被执行*/
        group.add(this);
        ...
    }
```

**run() 只是在当前线程中启动，start() 才是真正新建一个线程并启动。**

# 参考

[进程与线程的一个简单解释](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)

