---
title: 【Java并发编程】Java内存模型
toc: true
categories:
- Java
- Java并发编程
tags:
  - java
  - 并发编程
abbrlink: f7a60bf6
date: 2019-09-07 18:28:41
---

我们知道 CPU 与内存、IO 设备之间的读写速度存在巨大差距，短板理论也告诉我们计算机的性能取决于其性能最差的部分。

为了使内存和 IO 设备能够充分利用 CPU 的性能，同时也要兼顾设备的成本，操作系统和编译器为了做了许多优化，主要有 3 个方面：

1. CPU 增加缓存来平衡与内存之间速度的差异
2. 操作系统使用进程与线程，通过分时复用 CPU 来平衡 CPU 和 IO 设备速度的差异
3. 编译器通过优化指令执行次序，来合理利用 CPU 的缓存

当然，万事万物有利有弊，正式由于这些优化手段，也引入了并发编程的核心问题，即：

1. 缓存引起的可见性问题
2. 线程切换引起的原子性问题
3. 编译优化引起的有序性问题

以上就是并发编程的 3 大核心问题。对于Java并发编程这个话题，我们主要研究的就是怎么解决这 3 个问题。

解决的思路说来也不难，就是**按需禁用缓存和编译优化**。在我们需要的时候，程序员指定程序禁用缓存和编译优化就可以提升程序的性能。

这就是我们本文的重点**Java内存模型。**

![Java 内存模型](http://image.shuiyujie.com/2019-09-07-18-53-49.png)

Java 内存模型规范了 JVM 实现按需禁用缓存和编译优化的方法。其主要包括**volatile**、**synchronized **和  **final** 三个关键字，以及 6 项 **Happens-Before** 规则。

<!-- more -->

# Happens-Before

Happens-Before 是一种规则，它要求 JVM 按照这种规则来禁用内存和编译优化。简单来说 Happens-Before 规定了**前一个操作结果对后一个操作是可见的**。

举例来说现在有事件 A 和事件 B，且 A happens-before B，那就意味着事件 A 对于事件 B 是可见的，无论事件 A 和事件 B 发生在同一个线程还是不同线程，即使它们分别在两个不同的线程上发生，happens-before都会保证事件 A 对于事件 B 是可见的。具体包括以下 6 条规则：

1. 程序的顺序性规则
2. volatile 变量规则
3. 传递性规则
4. 管程中锁的规则
5. 线程的 start() 规则
6. 线程的 join() 规则

# 程序的顺序性规则

```java
public class VolatileExample {
    int x = 0;
    volatile boolean v = false;
    public void write(){
        x = 42;
        v = true;
    }
    
    public void read(){
        if(v){
            System.out.println(x);
        }
    }
}
```

程序的顺序性规则即，前面的操作 happens-before 后面的操作，比如 `x=10` 对于 `v=true` 是可见的。

# volatile 规则

volatile 在告诉编译器，对于这个变量的读写，不能使用 CPU 缓存，而是直接从内存执行读写操作。

volatile 规则规定，对于一个 volatile 变量的写操作，happens-before 于对这个变量的读操作，也就是上面这段代码的`write()`对于`read()`是可见的。

这条可以结合下面这条传递性规则来理解。

# 传递性规则

如果 A happens-before B，B happens-before A，则 A happens-before C。这条也比较符合直觉，结合 volatile 规则来看一下。

![传递性规则](http://image.shuiyujie.com/2019-09-07-20-39-07.png)

从图中可以看出：

1. 根据规则1，`x=42` happens-before `v=true`
2. 根据规则2，写变量 happens-before 读变量

再结合传递性规则，可得：`x=42` happens-before `读变量v=true`，这就意味着线程 B 可以看到`x==42`。

而在 JDK.5 之前，x 的值可能是 0，也可能是 42。因为共享变量 x，可能会缓存在 CPU 中。JDK1.5 对volatile 语义的增强，确保了线程 B 读取到的 x 肯定是 42。

# 管程中的锁的规则

这条规则是指对一个锁的解锁 Happens-Before 于后续对这个锁的加锁。

```java
synchronized(this) { // 此处自动加锁
	// x 是共享变量，初始值 = 10
  if(this.x < 12) {
    this.x = 12;
  }
}// 此处自动加锁
```

synchronized 是 Java 对于管程的实现，简单来说当代码进入同步块就会自动加锁，同步块执行完成就会释放锁。

将其与规则4结合是这样的：线程 A 执行代码块将 x 的值从 10 修改为 12；之后线程 B 进入代码块能够看到线程 A 对于 x 的写操作。

# 线程的 start() 规则

主线程 A 启动子线程 B 之后，子线程 B 能够看到主线程在启动子线程 B 前的操作。具体来说，主线程调用子线程的`start()`方法前的操作，子线程都能看到。

# 线程的 join() 规则

这条关于线程等待，在线程 A 中，调用线程 B 的 join() 并成功返回，那么线程 B 中的任意操作 Happens-Before 于该 join() 操作的返回。