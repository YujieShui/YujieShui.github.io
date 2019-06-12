---
title: STL-vector 容器
toc: true
abbrlink: 39bf26c3
date: 2019-06-07 23:42:53
categories:
  - C/C++
tags:
  - stl
  - c++
---

![vector](http://image.shuiyujie.com/2019-06-08-00-31-12.png)

- vector 是动态数组，连续内存空间，具有随机存取效率高的优点。
- vector 是单口容器，在队尾插入和删除元素效率高，在指定位置插入会导致数据 元素移动，效率低。

<!-- more -->

# vector 构造函数

```c++
vector<T> v; //采用模板实现类实现，默认构造函数
vector(v.begin(), v.end());//将 v[begin(), end())区间中的元素拷贝给本身。
vector(n, elem);//构造函数将 n 个 elem 拷贝给本身。
vector(const vector &vec);//拷贝构造函数。

//例子 使用第二个构造函数我们可以...
int arr[] = {2,3,4,1,9};
vector<int> v1(arr, arr + sizeof(arr)/sizeof(int));
```

# vector 常用赋值操作

```c++
assign(beg, end);//将[beg, end)区间中的数据拷贝赋值给本身。
assign(n, elem);//将 n 个 elem 拷贝赋值给本身。
vector& operator=(const vector &vec);//重载等号操作符
swap(vec);// 将 vec 与本身的元素互换。

//第一个赋值函数，可以这么写：
int arr[] = { 0, 1, 2, 3, 4 };
assign(arr, arr + 5);//使用数组初始化 vector
```

# vector 大小操作

```c++
size();//返回容器中元素的个数
empty();//判断容器是否为空
resize(int num);//重新指定容器的长度为 num，若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的元素被删除。
resize(int num, elem);//重新指定容器的长度为 num，若容器变长，则以 elem 值填充新位置。如果容器变短，则末尾超出容器长>度的元素被删除。
capacity();//容器的容量
reserve(int len);//容器预留 len个元素长度，预留位置不初始化，元素不可访问。
```

- resize 若容器变长，则以默认值填充新位置。如果容器变短，则末尾超出容器长度的 元素被删除。
- reserve 是容器预留空间，但在空间内不真正创建元素对象，所以在没有添加新的对 象之前，不能引用容器内的元素.
- resize 是改变容器的大小，且在创建对象，因此，调用这个函数之后，就可以引用容器内的对象了.

# vector 数据存取操作

```c++
at(int idx); //返回索引 idx 所指的数据，如果 idx 越界，抛出 out_of_range 异常。
operator[];//返回索引 idx 所指的数据，越界时，运行直接报错
front();//返回容器中第一个数据元素
back();//返回容器中最后一个数据元素
```

# vector 插入和删除操作

```c++
insert(const_iterator pos, int count,ele);//迭代器指向位置 pos 插入 count 个元素 ele.
push_back(ele); //尾部插入元素 ele
pop_back();//删除最后一个元素
erase(const_iterator start, const_iterator end);//删除迭代器从 start 到 end 之间的元素
erase(const_iterator pos);//删除迭代器指向的元素
clear();//删除容器中所有元素
```

# 总结

vector 是个动态数组，当空间不足的时候插入新元素，vector 会重新申请一块更大的 内存空间，将旧空间数据拷贝到新空间，然后释放旧空间。vector 是单口容器，所以在尾 端插入和删除元素效率较高，在指定位置插入，势必会引起数据元素移动，效率较低。

