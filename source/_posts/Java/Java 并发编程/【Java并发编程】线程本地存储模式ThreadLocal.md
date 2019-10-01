---
title: 【Java并发编程】线程本地存储模式ThreadLocal
toc: true
categories:
- Java
- Java并发编程
tags:
  - java
  - 并发编程
  - 设计模式
abbrlink: 790a1fa3
date: 2019-09-16 22:29:45
---

我们知道多个线程同时读写同一**共享变量**会导致并发问题。

一种解决方案是使用 Immutability 模式，如果共享变量在初始化之后就不会改变，只能读取，那么无论多少个线程同时读这个共享变量都不会出现并发问题。比如说 Java 中的 Long、Integer、Short、Byte 等基本数据类型的包装类的实现。

另一种解决方案是突破共享变量，**没有共享变量就不会有并发问题**。那么如何避免共享呢？思路其实很简单，就是每个线程拥有自己的变量，彼此不共享，就不会有共享问题。

具体来说有两种方法：**线程封闭**和线程本地存储(**ThreadLocal**)。

<!-- more -->

# 线程封闭

方法里的局部变量，因为不会和其他线程共享，所以没有并发问题，叫做**线程封闭**，比较官方的解释是：**仅在单线程内访问数据**。由于不存在共享，所以即便不同步也不会有并发问题，性能杠杠的。

![保护局部变量的调用栈结构](http://image.shuiyujie.com/2019-09-16-22-32-43.png)

如上图所示，JVM 中每一个线程都会有一个 **Java 虚拟机栈**。Java 程序每调用一个方法都会入栈，每执行完一个方法都会出栈，且每一个方法都有一个**栈帧**。栈帧中存储了参数、局部变量和返回地址。由此可见，方法的局部变量是不可能和其它线程共享的。

局部变量可以避免线程共享，此外还有什么方法能避免线程共享么？有，那就是 Java 语言提供的线程本地存储(ThreadLocal)。

# ThreadLock

SimpleDateFormat 不是线程安全的，如果想在并发场景下使用它可以将其设置为一个方法的局部变量，这样就会创建许多对象占用内存。或者使用 ThreadLock，不同线程调用 `SafeDateFormat` 会返回不同的 SimpleDateFormat 对象，但是由于在线程之间不共享，就和局部变量一样是安全的。

```java
static class SafeDateFormat {
  // 定义 ThreadLocal 变量
  static final ThreadLocal<DateFormat>
    tl=ThreadLocal.withInitial(
    ()-> new SimpleDateFormat(
      "yyyy-MM-dd HH:mm:ss"));

  static DateFormat get(){
    return tl.get();
  }
}
```

接下来再看一下 ThreadLock 的代码实现。

```java
class Thread {
  // 内部持有 ThreadLocalMap
  ThreadLocal.ThreadLocalMap 
    threadLocals;
}
class ThreadLocal<T>{
  public T get() {
    // 首先获取线程持有的
    //ThreadLocalMap
    ThreadLocalMap map =
      Thread.currentThread()
        .threadLocals;
    // 在 ThreadLocalMap 中
    // 查找变量
    Entry e = 
      map.getEntry(this);
    return e.value;  
  }
  static class ThreadLocalMap{
    // 内部是数组而不是 Map
    Entry[] table;
    // 根据 ThreadLocal 查找 Entry
    Entry getEntry(ThreadLocal key){
      // 省略查找逻辑
    }
    //Entry 定义
    static class Entry extends
    WeakReference<ThreadLocal>{
      Object value;
    }
  }
}
```

在 Java 的实现方案中，ThreadLocal 仅仅是一个工具类，内部并不持有和线程有关的数据，所有和线程有关的数据都存储在 Thread 中，比如 Thread 内部持有当前线程的 ThreadLocalMap。

这样设计有一个好处就是能够**避免内存泄露**。



[SimpleDateFormat 的线程安全问题与 ThreadLocal](https://blog.jrwang.me/2016/java-simpledateformat-multithread-threadlocal/)

[手撕面试题ThreadLocal！！！](http://ifeve.com/%e6%89%8b%e6%92%95%e9%9d%a2%e8%af%95%e9%a2%98threadlocal%ef%bc%81%ef%bc%81%ef%bc%81/)

