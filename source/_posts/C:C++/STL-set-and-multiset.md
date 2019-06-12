---
title: STL-set and multiset
toc: true
categories:
  - C/C++
tags:
  - stl
  - c++
abbrlink: adaa2861
date: 2019-06-08 12:15:00
---

set 的底层是红黑树。具有良好的查找效率。set 容器中不允许出现重复的元素，multiset 允许重复元素。

<!-- more -->

# set 构造函数

```c++
set<T> st;//set 默认构造函数：
mulitset<T> mst; //multiset 默认构造函数:
set(const set &st);//拷贝构造函数
```

# set 赋值操作

```c++
set& operator=(const set &st);//重载等号操作符 
swap(st);//交换两个集合容器
```

# set 大小操作

```c++
size();//返回容器中元素的数目
empty();//判断容器是否为空
```

# set 插入和删除操作

```c++
insert(elem);//在容器中插入元素。
clear();//清除所有元素
erase(pos);//删除 pos 迭代器所指的元素，返回下一个元素的迭代器。
erase(beg, end);//删除区间[beg,end)的所有元素 ，返回下一个元素的迭代器。
erase(elem);//删除容器中值为 elem 的元素。
```

# set 查找操作

```c++
find(key);//查找键 key 是否存在,若存在， 返回该键的元素的迭代器；若不存在， 返回 map.end();
lower_bound(keyElem);//返回第一个 key>=keyElem 元素的迭代器。
upper_bound(keyElem);//返回第一个 key>keyElem 元素的迭代器。
equal_range(keyElem);//返回容器中 key 与 keyElem相等的上下限的两个迭代器。
```

