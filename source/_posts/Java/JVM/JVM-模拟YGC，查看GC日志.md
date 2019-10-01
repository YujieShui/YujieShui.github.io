---
title: JVM-模拟YGC，查看GC日志
toc: true
categories:
  - Java
  - JVM
tags:
  - java
  - jvm
abbrlink: 5dbc4dfe
date: 2019-09-14 11:06:20
---

我们知道每次创建新的对象都会保存到**新生代**中。新生代会使用垃圾回收器比如说 ParNew 垃圾回收器，将新生代进一步分成 Eden 区和两个 Survivor 区。当新生代满了的时候就会触发 Young GC。

本文会实战 Young GC 的场景，并且带大家查看 GC 日志。

<!-- more -->

# 模拟 Young GC 场景

首先我们将 JVM 的参数按如下所示设置：

```
-XX:NewSize=5242880 -XX:MaxNewSize=5242880 -XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:SurvivorRatio=8 -XX:PretenureSizeThreshold=10485760 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log
```

以上参数都是 JDK1.8 版本的参数，其他版本略有不同，但也没差多少：

| 参数                                         | 含义                           |
| -------------------------------------------- | ------------------------------ |
| `-XX:InitialHeapSize`<br />`-XX:MaxHeapSize` | 初始堆大小和最大堆大小         |
| `-XX:NewSize`<br />`-XX:MaxNewSize`          | 初始新生代大小和最大新生代大小 |
| `-XX:PretenureSizeThreshold=1048576`         | 指定了大对象阈值是10MB         |
| `-XX:+PrintGCDetils`                         | 打印详细的gc日志               |
| `-XX:+PrintGCTimeStamps`                     | 打印出来每次GC发生的时间       |
| `-Xloggc:gc.log`                             | 设置将gc日志写入一个磁盘文件   |



![内存分配情况](http://image.shuiyujie.com/2019-09-25-21-57-16.png)