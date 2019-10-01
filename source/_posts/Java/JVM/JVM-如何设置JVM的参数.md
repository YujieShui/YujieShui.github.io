---
title: JVM-如何设置JVM的参数
toc: true
categories:
  - Java
  - JVM
tags:
  - java
  - jvm
abbrlink: 1410ce3d
date: 2019-09-13 15:58:49
---

1. 新生代、老生代、永久代的概念
2. JVM内存相关核心参数图解
3. 如何在启动系统的时候设置 JVM 参数

<!-- more -->

# 新生代、老生代、永久代的概念

[JVM-JVM中内存区域划分](https://shuiyujie.com/post/3860d841.html#more)这一章中讲过，每执行一个方法，该方法都会有一个栈帧进入 Java 虚拟机栈，栈帧中保存着方法中的局部变量、返回值地址等信息。

栈帧中保存的局部变量如果是一个对象，它就会指向 Java 堆中某个具体的对象实例，如下图所示：

![Java虚拟机栈和 Java 堆](http://image.shuiyujie.com/2019-09-13-16-10-29.png)

- 新生代：刚创建的对象都会在新生代，变量将会指向这个对象实例。如果没有任何变量指向这个对象，该对象就可能被回收。
- 老年代：新生代内存满了之后就会触发 `Minor GC` ，垃圾回收器会回收新生代中没有人引用的对象实例。如果一个对象实例经过10多次回收都没有被回收掉，就算它年龄有 10 多岁了，将会进入到老年代。
- 永久代：可以理解成方法区中的类和类信息

# JVM内存相关核心参数图解

在JVM内存分配中的核心参数，如下所示。

1. **-Xms**：Java堆内存的大小
2. **-Xmx**：Java堆内存的最大大小
3. **-Xmn**：Java堆内存中的新生代大小，扣除新生代剩下的就是老年代的内存大小了
4. **-XX:PermSize**：永久代大小
5. **-XX:MaxPermSize**：永久代最大大小
6. **-Xss**：每个线程的栈内存大小

![JVM内存参数](http://image.shuiyujie.com/2019-09-13-16-42-52.png)

# 启动时设置 JVM 参数

IDEA 可以在 `Run-edit configration..` 中设置参数，如下所示：

![Java内存参数设置](http://image.shuiyujie.com/Java内存参数设置.png)

如果在线上部署系统，可以使用`java -jar`方式启动，如：`java -Xms512M -Xmx512M -Xmn256M -Xss1M -XX:PermSize=128M -XX:MaxPermSize=128M -jar App.jar`


