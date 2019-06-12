---
title: STL-stack 容器
toc: true
categories:
  - C/C++
tags:
  - stl
  - c++
abbrlink: ac4a7cfa
date: 2019-06-08 11:53:08
---

栈 stack 是一种先进后出(first in last out,FILO)的数据结构，它只有一个出口，stack 只允许在栈顶新增元素，移除元素。

![栈](http://image.shuiyujie.com/2019-06-08-11-56-29.png)

栈不能遍历,不支持随机存取，只能通过 top 从栈顶获取和删除元素。

<!-- more -->

# stack 构造函数

```c++
stack<T> stkT;//stack 采用模板类实现, stack 对象的默认构造形式：
stack(const stack &stk);//拷贝构造函数
```

# stack 赋值操作

```c++
stack& operator=(const stack &stk);//重载等号操作符
```

# stack 数据存取操作

```c++
push(elem);//向栈顶添加元素
pop();//从栈顶移除第一个元素
top();//返回栈顶元素
```

# stack 大小操作

```c++
empty();//判断堆栈是否为空
size();//返回堆栈的大小
```

