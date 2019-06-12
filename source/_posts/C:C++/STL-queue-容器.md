---
title: STL-queue 容器
toc: true
abbrlink: 4244b416
date: 2019-06-07 23:43:13
categories:
  - C/C++
tags:
  - stl
  - c++
---

queue 是一种先进先出(first in first out, FIFO)的数据类型,他有两个口，只允许从队尾插入，队头弹出。

![queue](http://image.shuiyujie.com/2019-06-08-12-04-42.png)

- 必须从一个口数据元素入队，另一个口数据元素出队。
- 不能随机存取，不支持遍历

<!-- more -->

# queue 构造函数

```c++
queue<T> queT;//queue 采用模板类实现，queue 对象的默认构造形式：
queue(const queue &que);//拷贝构造函数
```

# queue 存取、插入和删除操作

```c++
push(elem);//往队尾添加元素
pop();//从队头移除第一个元素
back();//返回最后一个元素
front();//返回第一个元素
```

# queue 赋值操作

```c++
queue& operator=(const queue &que);//重载等号操作符
```

# queue 大小操作

```c++
empty();//判断队列是否为空
size();//返回队列的大小
```

