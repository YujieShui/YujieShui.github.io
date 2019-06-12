---
title: STL-deque 容器
toc: true
abbrlink: 564ccba3
date: 2019-06-07 23:43:04
categories:
  - C/C++
tags:
  - stl
  - c++
---

deque 是 “double-ended queue”的缩写,和 vector 一样，deque 也支持随机存取。 与 vector 不同的是，vector 是单向开口的，deque 是双向开口的，在两端进行插入和删除操作的时间复杂度为 O(1)。此外，deque 没有容量的概念，因为它是动态的以分段的连续空间组合而成，随时可 以增加一段新的空间并链接起来，换句话说，像 vector 那样“因旧空间不足而重新分配一 块更大的空间，然后再复制元素，释放空间”这样的操作不会发生在 deque 身上，也因此 deque 没有必要提供所谓的空间保留功能。

![deque操作示意图](http://image.shuiyujie.com/2019-06-08-09-17-46.png)

- 双端插入和删除元素效率较高. 
- 指定位置插入也会导致数据元素移动,降低效率. 
- 可随机存取,效率高.

<!-- more -->

# deque 构造函数

```c++
deque<T> deqT;//默认构造形式
deque(beg, end);//构造函数将[beg, end)区间中的元素拷贝给本身。
deque(n, elem);//构造函数将 n 个 elem 拷贝给本身。
deque(const deque &deq);//拷贝构造函数。
```

# deque 赋值操作

```c++
assign(beg, end);//将[beg, end)区间中的数据拷贝赋值给本身。
assign(n, elem);//将 n 个 elem 拷贝赋值给本身。
deque& operator=(const deque &deq); //重载等号操作符
swap(deq);// 将 deq 与本身的元素互换
```

# deque 大小操作

```c++
deque.size();//返回容器中元素的个数
deque.empty();//判断容器是否为空
deque.resize(num);//重新指定容器的长度为 num,若容器变长，则以默认值填充新位置。如果容器 变短，则末尾超出容器长度的元素被删除。
deque.resize(num, elem); //重新指定容器的长度为 num,若容器变长，则以 elem 值填充新位置,如果容器变短，则末尾超出容器长度的元素被删除。
```

# deque 双端插入和删除操作

```c++
push_back(elem);//在容器尾部添加一个数据
push_front(elem);//在容器头部插入一个数据
pop_back();//删除容器最后一个数据
pop_front();//删除容器第一个数据
```

# deque 数据存取

```c++
at(idx);//返回索引 idx 所指的数据，如果 idx 越界，抛出 out_of_range。
operator[];//返回索引 idx 所指的数据，如果 idx 越界，不抛出异常，直接出错。
front();//返回第一个数据。
back();//返回最后一个数据
```

# deque 插入操作

```c++
insert(pos,elem);//在 pos 位置插入一个 elem 元素的拷贝，返回新数据的位置。
insert(pos,n,elem);//在 pos 位置插入 n 个 elem 数据，无返回值。
insert(pos,beg,end);//在 pos 位置插入[beg,end)区间的数据，无返回值。
```

deque 是分段连续的内存空间，通过中控器维持一种连续内存空间的状态， 其实现复杂性要大于 vector queue stack 等容器，其迭代器的实现也更加复杂。

在需要对 deque 容器元素进行排序的时候，建议先将 deque 容器中数据数据元素拷贝到 vector 容 器中，对 vector 进行排序，然后再将排序完成的数据拷贝回 deque 容器。

# deque 删除操作

```c++
clear();//移除容器的所有数据
erase(beg,end);//删除[beg,end)区间的数据，返回下一个数据的位置。
erase(pos);//删除 pos 位置的数据，返回下一个数据的位置。
```

