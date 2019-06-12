---
title: STL-list 容器
toc: true
categories:
  - C/C++
tags:
  - stl
  - c++
abbrlink: b0cfebdb
date: 2019-06-08 12:13:44
---

链表通过数据域保存元素，通过指针域表示相邻元素之间的关系。

![list](http://image.shuiyujie.com/2019-06-08-12-18-30.png)

- 采用动态存储分配，不会造成内存浪费和溢出
- 链表执行插入和删除操作的效率高
- 链表灵活，但是空间和时间额外耗费较大

<!-- more -->

# list 构造函数

```c++
list<T> lstT;//list 采用采用模板类实现,对象的默认构造形式：
list(beg,end);//构造函数将[beg, end)区间中的元素拷贝给本身。
list(n,elem);//构造函数将 n 个 elem 拷贝给本身。
list(const list &lst);//拷贝构造函数。
```

# list 数据元素插入和删除操作

```c++
push_back(elem);//在容器尾部加入一个元素
pop_back();//删除容器中最后一个元素
push_front(elem);//在容器开头插入一个元素
pop_front();//从容器开头移除第一个元素
insert(pos,elem);//在 pos 位置插 elem 元素的拷贝，返回新数据的位置。
insert(pos,n,elem);//在 pos 位置插入 n 个 elem 数据，无返回值。
insert(pos,beg,end);//在 pos 位置插入[beg,end)区间的数据，无返回值。
clear();//移除容器的所有数据
erase(beg,end);//删除[beg,end)区间的数据，返回下一个数据的位置。
erase(pos);//删除 pos 位置的数据，返回下一个数据的位置。
remove(elem);//删除容器中所有与 elem 值匹配的元素。
```

# list 大小操作

```c++
size();//返回容器中元素的个数
empty();//判断容器是否为空
resize(num);//重新指定容器的长度为 num，若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
resize(num, elem);//重新指定容器的长度为 num， 若容器变长，则以 elem 值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
```

# list 赋值操作

```c++
assign(beg, end);//将[beg, end)区间中的数据拷贝赋值给本身。
assign(n, elem);//将 n 个 elem 拷贝赋值给本身。
list& operator=(const list &lst);//重载等号操作符
swap(lst);//将 lst 与本身的元素互换。
```

# list 数据的存取

```c++
front();//返回第一个元素。
back();//返回最后一个元素。
```

# list 反转排列排序

```c++
reverse();//反转链表，比如 lst 包含 1,3,5 元素，运行此方法后， lst 就包含 5,3,1 元素。
sort(); //list 排序
```

